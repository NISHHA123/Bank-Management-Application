#include <iostream>
#include <fstream>
#include <cstring>
using namespace std;

class BankAccount {
private:
    int accountNumber;
    char name[50];
    double balance;

public:
    void createAccount() {
        cout << "Enter Account Number: ";
        cin >> accountNumber;

        cin.ignore();
        cout << "Enter Customer Name: ";
        cin.getline(name, 50);

        cout << "Enter Initial Balance: ";
        cin >> balance;
    }

    void displayAccount() const {
        cout << "\nAccount Number : " << accountNumber;
        cout << "\nCustomer Name  : " << name;
        cout << "\nBalance        : " << balance << endl;
    }

    int getAccountNumber() const {
        return accountNumber;
    }

    double getBalance() const {
        return balance;
    }

    void deposit(double amount) {
        balance += amount;
    }

    bool withdraw(double amount) {
        if (amount > balance) {
            return false;
        }
        balance -= amount;
        return true;
    }
};

// Create New Account
void createNewAccount() {
    BankAccount acc;

    ofstream file("bank.dat", ios::binary | ios::app);

    acc.createAccount();

    file.write(reinterpret_cast<char*>(&acc), sizeof(acc));
    file.close();

    cout << "\nAccount Created Successfully!\n";
}

// Display Account
void displayAccount(int accNo) {
    BankAccount acc;
    bool found = false;

    ifstream file("bank.dat", ios::binary);

    while (file.read(reinterpret_cast<char*>(&acc), sizeof(acc))) {
        if (acc.getAccountNumber() == accNo) {
            acc.displayAccount();
            found = true;
            break;
        }
    }

    file.close();

    if (!found)
        cout << "\nAccount Not Found!\n";
}

// Deposit Money
void depositMoney(int accNo, double amount) {
    BankAccount acc;
    bool found = false;

    fstream file("bank.dat", ios::binary | ios::in | ios::out);

    while (file.read(reinterpret_cast<char*>(&acc), sizeof(acc))) {
        if (acc.getAccountNumber() == accNo) {
            acc.deposit(amount);

            int pos = file.tellg();
            file.seekp(pos - sizeof(acc));

            file.write(reinterpret_cast<char*>(&acc), sizeof(acc));

            cout << "\nDeposit Successful!\n";
            found = true;
            break;
        }
    }

    file.close();

    if (!found)
        cout << "\nAccount Not Found!\n";
}

// Withdraw Money
void withdrawMoney(int accNo, double amount) {
    BankAccount acc;
    bool found = false;

    fstream file("bank.dat", ios::binary | ios::in | ios::out);

    while (file.read(reinterpret_cast<char*>(&acc), sizeof(acc))) {
        if (acc.getAccountNumber() == accNo) {

            if (acc.withdraw(amount))
                cout << "\nWithdrawal Successful!\n";
            else {
                cout << "\nInsufficient Balance!\n";
                file.close();
                return;
            }

            int pos = file.tellg();
            file.seekp(pos - sizeof(acc));

            file.write(reinterpret_cast<char*>(&acc), sizeof(acc));

            found = true;
            break;
        }
    }

    file.close();

    if (!found)
        cout << "\nAccount Not Found!\n";
}

// Main Menu
int main() {
    int choice, accNo;
    double amount;

    do {
        cout << "\n========== BANK MANAGEMENT SYSTEM ==========\n";
        cout << "1. Create Account\n";
        cout << "2. Deposit Money\n";
        cout << "3. Withdraw Money\n";
        cout << "4. Balance Inquiry\n";
        cout << "5. Exit\n";
        cout << "Enter Choice: ";
        cin >> choice;

        switch (choice) {

        case 1:
            createNewAccount();
            break;

        case 2:
            cout << "Enter Account Number: ";
            cin >> accNo;
            cout << "Enter Deposit Amount: ";
            cin >> amount;
            depositMoney(accNo, amount);
            break;

        case 3:
            cout << "Enter Account Number: ";
            cin >> accNo;
            cout << "Enter Withdrawal Amount: ";
            cin >> amount;
            withdrawMoney(accNo, amount);
            break;

        case 4:
            cout << "Enter Account Number: ";
            cin >> accNo;
            displayAccount(accNo);
            break;

        case 5:
            cout << "\nThank You for Using Bank Management System.\n";
            break;

        default:
            cout << "\nInvalid Choice!\n";
        }

    } while (choice != 5);

    return 0;
}
