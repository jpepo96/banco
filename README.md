import hashlib
import uuid

# Classe para gerenciar contas
class Account:
    def __init__(self, username, password):
        self.username = username
        self.password_hash = self.hash_password(password)
        self.balance = 0.0
        self.transactions = []

    def hash_password(self, password):
        salt = uuid.uuid4().hex
        return hashlib.sha256(salt.encode() + password.encode()).hexdigest() + ':' + salt

    def check_password(self, password):
        password_hash, salt = self.password_hash.split(':')
        return password_hash == hashlib.sha256(salt.encode() + password.encode()).hexdigest()

    def __str__(self):
        return f"Account(username={self.username}, balance={self.balance})"

# Serviço de autenticação
class AuthenticationService:
    def __init__(self):
        self.accounts = {}

    def register(self, username, password):
        if username in self.accounts:
            return "Username already exists."
        self.accounts[username] = Account(username, password)
        return "Account created successfully."

    def login(self, username, password):
        account = self.accounts.get(username)
        if account and account.check_password(password):
            return "Login successful."
        return "Invalid username or password."

    def get_account(self, username):
        return self.accounts.get(username, None)

# Serviço de transações
class TransactionService:
    def __init__(self, auth_service):
        self.auth_service = auth_service

    def deposit(self, username, amount):
        account = self.auth_service.accounts.get(username)
        if account:
            account.balance += amount
            account.transactions.append(f"Deposit: {amount}")
            return f"Deposited {amount} to {username}'s account."
        return "Account not found."

    def withdraw(self, username, amount):
        account = self.auth_service.accounts.get(username)
        if account and account.balance >= amount:
            account.balance -= amount
            account.transactions.append(f"Withdraw: {amount}")
            return f"Withdrew {amount} from {username}'s account."
        return "Insufficient funds or account not found."

    def transfer(self, from_username, to_username, amount):
        from_account = self.auth_service.accounts.get(from_username)
        to_account = self.auth_service.accounts.get(to_username)
        if from_account and to_account and from_account.balance >= amount:
            from_account.balance -= amount
            to_account.balance += amount
            from_account.transactions.append(f"Transfer to {to_username}: {amount}")
            to_account.transactions.append(f"Transfer from {from_username}: {amount}")
            return f"Transferred {amount} from {from_username} to {to_username}."
        return "Insufficient funds or account not found."

# Função principal para demonstrar o funcionamento do sistema
def main():
    auth_service = AuthenticationService()
    transaction_service = TransactionService(auth_service)

    while True:
        print("\n1. Registrar")
        print("2. Login")
        print("3. Depositar")
        print("4. Sacar")
        print("5. Transferir")
        print("6. Ver detalhes da conta")
        print("7. Sair")
        choice = input("Enter choice: ")

        if choice == '1':
            username = input("Enter username: ")
            password = input("Enter password: ")
            print(auth_service.register(username, password))
        
        elif choice == '2':
            username = input("Enter username: ")
            password = input("Enter password: ")
            print(auth_service.login(username, password))
        
        elif choice == '3':
            username = input("Enter username: ")
            amount = float(input("Enter amount to depositar: "))
            print(transaction_service.deposit(username, amount))
        
        elif choice == '4':
            username = input("Enter username: ")
            amount = float(input("Enter amount to sacar: "))
            print(transaction_service.withdraw(username, amount))
        
        elif choice == '5':
            from_username = input("Enter your username: ")
            to_username = input("Enter recipient's username: ")
            amount = float(input("Enter amount to transferir: "))
            print(transaction_service.transfer(from_username, to_username, amount))
        
        elif choice == '6':
            username = input("Enter username: ")
            account = auth_service.get_account(username)
            if account:
                print(f"Username: {account.username}")
                print(f"Balance: {account.balance}")
                print(f"Transactions: {account.transactions}")
            else:
                print("Account not found.")
        
        elif choice == '7':
            break

        else:
            print("Invalid choice. Please try again.")

if __name__ == "__main__":
    main()
