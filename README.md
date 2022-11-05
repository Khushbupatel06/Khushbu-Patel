#include<iostream>
#include<fstream>
#include<cstdlib>
#include<vector>
#include<map>
using namespace std;

class Account
{
    long accountNumber;
    string firstName;
    string lastName;
    float balance;
    static long NextAccountNumber;
public:
    Account(){};
    Account(string fname,string lname, float balance);
    long getAccNo() {return accountNumber;}
    string getFirstName() {return firstName;}
    string getLastName() {return lastName;}
    float getBalance() {return balance;}
    
    void Deposit(float amount);
    void Withdraw(float amount);
    static void setLastAccountNumber(long accountNumber);
    static long getLastAccountNumber();
    friend ofstream & operator<<(ofstream &ofs, Account &acc);
    friend ifstream & operator>>(ifstream &ifs, Account &acc);
    friend ostream & operator>>(ostream &os, Account &acc);
};

long Account::NextAccountNumber=0;

class Bank
{
private:
    map<long,Account>accounts;
public:
    Bank();
    Account OpenNewAccount(string fname,string lname, float balance);
    float CheckBalance(long accountNumber);
    Account Deposit(long accountNumber, float amount);
    Account Withdraw(long accountNumber, float amount);
    void CloseAccount(long accountNamber);
    void ShowAllAccounts();
    ~Bank();
};

int main()
{
    Bank b;
    Account acc;
    int option;
    string fname, lname;
    long accountNumber;
    float balance;
    float amount;
    cout<<"Welcome to Banking System"<<endl;
    do 
    {
        cout<<"\n\tSelect one option from below.";
        cout<<"\n\t1. Open New Account";
        cout<<"\n\t2. Check Balance in your Account";
        cout<<"\n\t3. Deposit Amount";
        cout<<"\n\t4. Withdraw Amount";
        cout<<"\n\t5. Close your Account";
        cout<<"\n\t6. Show all Accounts";
        cout<<"\n\t7. Exit"<<endl;
        cout<<"\nEnter your Option: ";
        cin>>option;
        
        switch(option)
        {
            case 1:
                cout<<"Enter your First Name: ";
                cin>>fname;
                cout<<"Enter your Last Name: ";
                cin>>lname;
                cout<<"Enter Initial Balance: ";
                cin>>balance;
                acc=b.OpenNewAccount(fname, lname, balance);
                cout<<"Congratulation your Account is Created "<<endl;
                break;
            case 2:
                cout<<"Enter your Account Number: ";
                cin>>accountNumber;
                cout<<"Your Account Balance is: "<<b.CheckBalance(accountNumber)<<endl;
                break;
            case 3:
                cout<<"Enter your Account Number: ";
                cin>>accountNumber;
                cout<<"Enter the Amount to deposit: ";
                cin>>amount;
                acc=b.Deposit(accountNumber, amount);
                cout<<"Amount is Deposited."<<endl;
                cout<<"Your new Balance is: "<<b.CheckBalance(accountNumber)<<endl;
                break;
            case 4:
                cout<<"Enter your Account Number: ";
                cin>>accountNumber;
                cout<<"Enter the Amount to withdraw: ";
                cin>>amount;
                acc=b.Withdraw(accountNumber, amount);
                cout<<"Amount is withdrawn."<<endl;
                cout<<"Your new Balance is: "<<b.CheckBalance(accountNumber)<<endl;
                break;
            case 5:
                cout<<"Enter your Account Number: ";
                cin>>accountNumber;
                b.CloseAccount(accountNumber);
                break;
            case 6:
                b.ShowAllAccounts();
                break;
            case 7:break;
            default:
                cout<<"\nEnter Correct option";
                exit(0);
        }
    }while (option!=7);
    
    return 0;
}

Account::Account(string fname, string lname, float balance)
{
    NextAccountNumber++;
    accountNumber=NextAccountNumber;
    firstName=fname;
    lastName=lname;
    this->balance=balance;
}

void Account::Deposit(float amount)
{
    balance+=amount;
}

void Account::Withdraw(float amount)
{
    balance-=amount;
}

void Account::setLastAccountNumber(long accountNumber)
{
    NextAccountNumber=accountNumber;
}

long Account::getLastAccountNumber()
{
    return NextAccountNumber;
}

//writting data in file.//

ofstream & operator<<(ofstream &ofs, Account &acc)
{
    ofs<<acc.accountNumber<<endl;
    ofs<<acc.firstName<<endl;
    ofs<<acc.lastName<<endl;
    ofs<<acc.balance<<endl;
    return ofs;
}

//Reading data from file.//

ifstream & operator>>(ifstream & ifs, Account &acc)
{
    ifs>>acc.accountNumber;
    ifs>>acc.firstName;
    ifs>>acc.lastName;
    ifs>>acc.balance;
    return ifs;
}

//For Displaying Account Details.//

ostream & operator<<(ostream &os, Account &acc)
{
    os<<"First Name: "<<acc.getFirstName()<<endl;
    os<<"Last Name: "<<acc.getLastName()<<endl;
    os<<"Account Number: "<<acc.getAccNo()<<endl;
    os<<"New Balance is: "<<acc.getBalance()<<endl;
    return os;
}

Bank::Bank()
{
    Account account;
    ifstream infile;
    infile.open("Bank.data");
    if(!infile)
    {
        cout<<"Error in Opening! File is not Found!!"<<endl;
        return;
    }
    while(!infile.eof())
    {
        infile>>account;
        accounts.insert(pair<long, Account>(account.getAccNo(),account));
    }
    Account::setLastAccountNumber(account.getAccNo());
    
    infile.close();
}

//function for Opening Account.//

Account Bank::OpenNewAccount(string fname, string lname, float balance)
{
    ofstream outfile;
    Account account(fname, lname, balance);
    accounts.insert(pair<long,Account>(account.getAccNo(),account));
    
    outfile.open("Bank.data", ios::trunc);
    
    map<long,Account>::iterator itr;
    for(itr=accounts.begin();itr!=accounts.end();itr++)
    {
        outfile<<itr->second;
    }
    outfile.close();
    return account;
}

//function for checking Balance.//

float Bank::CheckBalance(long accountNumber)
{
    map<long,Account>::iterator itr=accounts.find(accountNumber);
    return itr->second.getBalance();
}

//function for Deposit amount.//

Account Bank::Deposit(long accountNumber,float amount)
{
    map<long,Account>::iterator itr=accounts.find(accountNumber);
    itr->second.Deposit(amount);
    return itr->second;
}

//function for Withdraw amount.//

Account Bank::Withdraw(long accountNumber, float amount)
{
    map<long,Account>::iterator itr=accounts.find(accountNumber);
    itr->second.Withdraw(amount);
    return itr->second;
}

//function for close Account.//

void Bank::CloseAccount(long accountNumber)
{
    map<long,Account>::iterator itr=accounts.find(accountNumber);
    cout<<"Your Account is Closed."<<endl;
    itr->second;
    accounts.erase(accountNumber);
}

//function for show All Account.//

void Bank::ShowAllAccounts()
{
    map<long,Account>::iterator itr;
    for(itr=accounts.begin();itr!=accounts.end();itr++)
    {
        cout<<"Account "<<itr->first<<endl<<itr->second<<endl;
    }
}

//function for exit.//

Bank::~Bank()
{
    ofstream outfile;
    outfile.open("Bank.data", ios::trunc);
    
    map<long,Account>::iterator itr;
    for(itr=accounts.begin(); itr!=accounts.end(); itr++)
    {
        outfile<<itr->second;
    }
    outfile.close();
}
