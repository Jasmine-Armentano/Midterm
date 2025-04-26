using System;
using System.Linq;

namespace PersonalBudgetTracker
{
    public enum TransactionType
    {
        Income, Expense
    }

    public class Transaction
    {
        public string Description { get; set; }
        public decimal Amount { get; set; }
        public TransactionType Type { get; set; }
        public string Category { get; set; }
        public DateTime Date { get; set; }

        public Transaction(string description, decimal amount, TransactionType type, string category, DateTime date)
        {
            Description = description;
            Amount = amount;
            Type = type;
            Category = category;
            Date = date;
        }
    }

    public class BudgetTracker
    {
        private Transaction[] transactions = new Transaction[100];
        private int transactionCount = 0;

        public void AddTransaction(Transaction transaction)
        {
            if (transactionCount < transactions.Length)
            {
                transactions[transactionCount] = transaction;
                transactionCount++;
                Console.WriteLine("\nTransaction successfully added!");
            }
            else
            {
                Console.WriteLine("Error: Cannot add more transactions. Array is full.");
            }
        }

        public void ViewAllTransactions()
        {
            Console.WriteLine("\n--- Transaction History ---");
            if (transactionCount == 0)
            {
                Console.WriteLine("No transactions recorded yet.");
                return;
            }

            for (int i = 0; i < transactionCount; i++)
            {
                var t = transactions[i];
                Console.WriteLine($"{i + 1}. {t.Date.ToShortDateString()} | {t.Type} | {t.Category} | {t.Description} | {t.Amount:C}");
            }
        }

        public decimal GetTotalIncome() => transactions
            .Where(t => t != null && t.Type == TransactionType.Income)
            .Sum(t => t.Amount);

        public decimal GetTotalExpenses() => transactions
            .Where(t => t != null && t.Type == TransactionType.Expense)
            .Sum(t => t.Amount);

        public decimal GetNetSavings() => GetTotalIncome() - GetTotalExpenses();

        public void DisplayTextGraph()
        {
            var expenses = transactions
                .Where(t => t != null && t.Type == TransactionType.Expense)
                .GroupBy(t => t.Category)
                .ToDictionary(g => g.Key, g => g.Sum(t => t.Amount));

            Console.WriteLine("\nExpense Chart by Category:");
            foreach (var item in expenses)
            {
                Console.WriteLine($"{item.Key.PadRight(15)}: {new string('*', (int)(item.Value / 10))} ({item.Value:C})");
            }
        }

        public void DisplayInsights()
        {
            var analytics = transactions
                .Where(t => t != null && t.Type == TransactionType.Expense)
                .GroupBy(t => t.Category)
                .ToDictionary(g => g.Key, g => g.Sum(t => t.Amount));

            if (analytics.Any())
            {
                var maxCategory = analytics.Aggregate((l, r) => l.Value > r.Value ? l : r);
                Console.WriteLine($"\nYour biggest expense is on **{maxCategory.Key}**, totaling {maxCategory.Value:C}.");
            }

            Console.WriteLine($"Estimated Monthly Savings: {GetNetSavings() / 12:C}");
        }
    }

    class Program
    {
        static readonly string[] Categories = { "Food", "Transport", "Utilities", "Entertainment", "Health", "Education", "Savings", "Others" };
        static readonly string[] DateOptions = { "Today" };

        static void Main(string[] args)
        {
            var tracker = new BudgetTracker();
            bool running = true;

            Console.WriteLine("Personal Budget Tracker!");

            while (running)
            {
                try
                {
                    Console.WriteLine("\n--- Main Menu ---");
                    Console.WriteLine("1. Add a Transaction");
                    Console.WriteLine("2. View Financial Summary");
                    Console.WriteLine("3. View Expense Analytics");
                    Console.WriteLine("4. View Transaction History");
                    Console.WriteLine("5. Exit");
                    Console.Write("Select an option (1-5): ");

                    string input = Console.ReadLine();
                    if (!int.TryParse(input, out int choice))
                    {
                        Console.WriteLine("Please enter a valid number.");
                        continue;
                    }

                    switch (choice)
                    {
                        case 1:
                            AddTransactionFlow(tracker);
                            break;

                        case 2:
                            if (Confirm("Would you like to see your summary?"))
                            {
                                Console.WriteLine($"\nTotal Income: {tracker.GetTotalIncome():C}");
                                Console.WriteLine($"Total Expenses: {tracker.GetTotalExpenses():C}");
                                Console.WriteLine($"Net Savings: {tracker.GetNetSavings():C}");
                            }
                            break;

                        case 3:
                            if (Confirm("Do you want a breakdown of your spending?"))
                            {
                                tracker.DisplayTextGraph();
                                tracker.DisplayInsights();
                            }
                            break;

                        case 4:
                            tracker.ViewAllTransactions();
                            break;

                        case 5:
                            if (Confirm("Are you sure you want to exit?"))
                                running = false;
                            break;

                        default:
                            Console.WriteLine("Oops! That option doesn't exist.");
                            break;
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Unexpected error: {ex.Message}");
                }
            }

            Console.WriteLine("Thanks for using the Budget Tracker. See you next time!");
        }

        static void AddTransactionFlow(BudgetTracker tracker)
        {
            if (!Confirm("Ready to add a new transaction?")) return;

            Console.Write("Enter a short description: ");
            string description = Console.ReadLine();

            decimal amount;
            while (true)
            {
                Console.Write("Enter the amount: ");
                if (decimal.TryParse(Console.ReadLine(), out amount) && amount >= 0) break;
                Console.WriteLine("Please enter a valid non-negative amount.");
            }

            TransactionType type;
            while (true)
            {
                Console.Write("Type (Income/Expense): ");
                string typeInput = Console.ReadLine();
                if (Enum.TryParse(typeInput, true, out type)) break;
                Console.WriteLine("Invalid type. Please enter 'Income' or 'Expense'.");
            }

            Console.WriteLine("Pick a Category:");
            for (int i = 0; i < Categories.Length; i++)
            {
                Console.WriteLine($"{i + 1}. {Categories[i]}");
            }

            int catIndex;
            while (true)
            {
                Console.Write("Choose (1-8): ");
                if (int.TryParse(Console.ReadLine(), out catIndex) && catIndex >= 1 && catIndex <= Categories.Length) break;
                Console.WriteLine("Please select a valid category number.");
            }

            DateTime date = DateTime.Today;
            Console.WriteLine($"Date: {date.ToShortDateString()}");

            var transaction = new Transaction(description, amount, type, Categories[catIndex - 1], date);
            tracker.AddTransaction(transaction);
        }

        static bool Confirm(string message)
        {
            Console.Write($"{message} (Y/N): ");
            string input = Console.ReadLine()?.Trim().ToLower();
            return input == "y" || input == "yes";
        }
    }
}
