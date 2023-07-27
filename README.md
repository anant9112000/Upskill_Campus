# Expense-tracker
My project is console-based expense tracker application  that allows users  to manage their personal expenses using Java. It include many functionalities like record &amp;track expenses, view spending summaries, Expense Recording, Expense Categories, Expense Tracking, Expense Filtering, Expense Modification and Deletion, Data Persistence and many more.
import java.io.*;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.*;

class Expense {
    private double amount;
    private String category;
    private String description;
    private LocalDateTime timestamp;

    public Expense(double amount, String category, String description) {
        this.amount = amount;
        this.category = category;
        this.description = description;
        this.timestamp = LocalDateTime.now();
    }

    public double getAmount() {
        return amount;
    }

    public String getCategory() {
        return category;
    }

    public String getDescription() {
        return description;
    }

    public LocalDateTime getTimestamp() {
        return timestamp;
    }
}

class ExpenseTracker {
    private List<Expense> expenses;
    private List<String> categories;
    private String dataFilePath;

    public ExpenseTracker(String dataFilePath) {
        this.dataFilePath = dataFilePath;
        expenses = new ArrayList<>();
        categories = new ArrayList<>();
        loadExpenses();
    }

    public void addExpense(Expense expense) {
        expenses.add(expense);
        saveExpenses();
    }

    public void removeExpense(Expense expense) {
        expenses.remove(expense);
        saveExpenses();
    }

    public List<Expense> getExpenses() {
        return expenses;
    }

    public List<String> getCategories() {
        return categories;
    }

    public double getTotalExpenses() {
        double total = 0;
        for (Expense expense : expenses) {
            total += expense.getAmount();
        }
        return total;
    }

    public List<Expense> getExpensesByCategory(String category) {
        List<Expense> categoryExpenses = new ArrayList<>();
        for (Expense expense : expenses) {
            if (expense.getCategory().equalsIgnoreCase(category)) {
                categoryExpenses.add(expense);
            }
        }
        return categoryExpenses;
    }

    public void addCategory(String category) {
        categories.add(category);
        saveExpenses();
    }

    public void removeCategory(String category) {
        categories.remove(category);
        // Remove the category from all expenses
        for (Expense expense : expenses) {
            if (expense.getCategory().equalsIgnoreCase(category)) {
                expense.setCategory("Uncategorized");
            }
        }
        saveExpenses();
    }

    private void loadExpenses() {
        try (BufferedReader reader = new BufferedReader(new FileReader(dataFilePath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] parts = line.split(",");
                if (parts.length == 4) {
                    double amount = Double.parseDouble(parts[0]);
                    String category = parts[1];
                    String description = parts[2];
                    String timestampStr = parts[3];
                    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
                    LocalDateTime timestamp = LocalDateTime.parse(timestampStr, formatter);
                    Expense expense = new Expense(amount, category, description);
                    expense.setTimestamp(timestamp);
                    expenses.add(expense);

                    if (!categories.contains(category)) {
                        categories.add(category);
                    }
                }
            }
        } catch (IOException e) {
            // Handle file loading exception
        }
    }

    private void saveExpenses() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(dataFilePath))) {
            for (Expense expense : expenses) {
                String line = expense.getAmount() + "," + expense.getCategory() + "," +
                        expense.getDescription() + "," + expense.getTimestamp().toString();
                writer.write(line);
                writer.newLine();
            }
        } catch (IOException e) {
            // Handle file saving exception
        }
    }
}

public class ExpenseTrackerApp {
    private static Scanner scanner = new Scanner(System.in);
    private static ExpenseTracker expenseTracker;

    public static void main(String[] args) {
        expenseTracker = new ExpenseTracker("expenses.txt");

        boolean exit = false;
        while (!exit) {
            displayMenu();
            int choice = scanner.nextInt();
            scanner.nextLine(); // Consume newline character

            switch (choice) {
                case 1:
                    recordExpense();
                    break;
                case 2:
                    viewSpendingSummary();
                    break;
                case 3:
                    manageCategories();
                    break;
                case 4:
                    filterExpensesByCategory();
                    break;
                case 5:
                    modifyExpense();
                    break;
                case 6:
                    deleteExpense();
                    break;
                case 7:
                    generateExpenseReport();
                    break;
                case 8:
                    exit = true;
                    System.out.println("Exiting Expense Tracker. Goodbye!");
                    break;
                default:
                    System.out.println("Invalid choice. Please try again.");
            }
        }
    }

    private static void displayMenu() {
        System.out.println("Expense Tracker Menu");
        System.out.println("1. Record Expense");
        System.out.println("2. View Spending Summary");
        System.out.println("3. Manage Categories");
        System.out.println("4. Filter Expenses by Category");
        System.out.println("5. Modify Expense");
        System.out.println("6. Delete Expense");
        System.out.println("7. Generate Expense Report");
        System.out.println("8. Exit");
        System.out.print("Enter your choice: ");
    }

    private static void recordExpense() {
        System.out.print("Enter amount: ");
        double amount = scanner.nextDouble();
        scanner.nextLine(); // Consume newline character

        System.out.print("Enter category: ");
        String category = scanner.nextLine();

        System.out.print("Enter description: ");
        String description = scanner.nextLine();

        Expense expense = new Expense(amount, category, description);
        expenseTracker.addExpense(expense);

        System.out.println("Expense recorded successfully.");
    }

    private static void viewSpendingSummary() {
        System.out.println("Total Expenses: " + expenseTracker.getTotalExpenses());

        System.out.print("Enter category to view expenses (leave blank to view all): ");
        String category = scanner.nextLine();

        List<Expense> expenses;
        if (category.isEmpty()) {
            expenses = expenseTracker.getExpenses();
        } else {
            expenses = expenseTracker.getExpensesByCategory(category);
        }

        if (expenses.isEmpty()) {
            System.out.println("No expenses found.");
        } else {
            System.out.println("Expenses:");
            for (Expense expense : expenses) {
                System.out.println("Amount: " + expense.getAmount());
                System.out.println("Category: " + expense.getCategory());
                System.out.println("Description: " + expense.getDescription());
                System.out.println("Timestamp: " + expense.getTimestamp());
                System.out.println("-------------------------");
            }
        }
    }

    private static void manageCategories() {
        boolean exit = false;
        while (!exit) {
            displayCategoriesMenu();
            int choice = scanner.nextInt();
            scanner.nextLine(); // Consume newline character

            switch (choice) {
                case 1:
                    addCategory();
                    break;
                case 2:
                    removeCategory();
                    break;
                case 3:
                    exit = true;
                    System.out.println("Exiting category management.");
                    break;
                default:
                    System.out.println("Invalid choice. Please try again.");
            }
        }
    }

    private static void displayCategoriesMenu() {
        System.out.println("Category Management");
        System.out.println("1. Add Category");
        System.out.println("2. Remove Category");
        System.out.println("3. Back to Main Menu");
        System.out.print("Enter your choice: ");
    }

    private static void addCategory() {
        System.out.print("Enter category name: ");
        String category = scanner.nextLine();
        expenseTracker.addCategory(category);
        System.out.println("Category added successfully.");
    }

    private static void removeCategory() {
        System.out.print("Enter category name: ");
        String category = scanner.nextLine();
        expenseTracker.removeCategory(category);
        System.out.println("Category removed successfully.");
    }

    private static void filterExpensesByCategory() {
        System.out.print("Enter category name: ");
        String category = scanner.nextLine();

        List<Expense> expenses = expenseTracker.getExpensesByCategory(category);
        if (expenses.isEmpty()) {
            System.out.println("No expenses found for the given category.");
        } else {
            System.out.println("Expenses:");
            for (Expense expense : expenses) {
                System.out.println("Amount: " + expense.getAmount());
                System.out.println("Category: " + expense.getCategory());
                System.out.println("Description: " + expense.getDescription());
                System.out.println("Timestamp: " + expense.getTimestamp());
                System.out.println("-------------------------");
            }
        }
    }

    private static void modifyExpense() {
        System.out.print("Enter the index of the expense to modify: ");
        int index = scanner.nextInt();
        scanner.nextLine(); // Consume newline character

        List<Expense> expenses = expenseTracker.getExpenses();
        if (index >= 0 && index < expenses.size()) {
            Expense expense = expenses.get(index);

            System.out.println("Old Amount: " + expense.getAmount());
            System.out.print("Enter new amount: ");
            double amount = scanner.nextDouble();
            scanner.nextLine(); // Consume newline character
            expense.setAmount(amount);

            System.out.println("Old Category: " + expense.getCategory());
            System.out.print("Enter new category: ");
            String category = scanner.nextLine();
            expense.setCategory(category);

            System.out.println("Old Description: " + expense.getDescription());
            System.out.print("Enter new description: ");
            String description = scanner.nextLine();
            expense.setDescription(description);

            expenseTracker.saveExpenses();
            System.out.println("Expense modified successfully.");
        } else {
            System.out.println("Invalid expense index.");
        }
    }

    private static void deleteExpense() {
        System.out.print("Enter the index of the expense to delete: ");
        int index = scanner.nextInt();
        scanner.nextLine(); // Consume newline character

        List<Expense> expenses = expenseTracker.getExpenses();
        if (index >= 0 && index < expenses.size()) {
            Expense expense = expenses.get(index);
            expenseTracker.removeExpense(expense);
            System.out.println("Expense deleted successfully.");
        } else {
            System.out.println("Invalid expense index.");
        }
    }

    private static void generateExpenseReport() {
        System.out.print("Enter report start date (yyyy-MM-dd): ");
        String startDateStr = scanner.nextLine();
        LocalDate startDate = LocalDate.parse(startDateStr);

        System.out.print("Enter report end date (yyyy-MM-dd): ");
        String endDateStr = scanner.nextLine();
        LocalDate endDate = LocalDate.parse(endDateStr);

        List<Expense> expenses = expenseTracker.getExpenses();
        double totalExpenses = 0;
        for (Expense expense : expenses) {
            LocalDate expenseDate = expense.getTimestamp().toLocalDate();
            if (expenseDate.isAfter(startDate) && expenseDate.isBefore(endDate)) {
                totalExpenses += expense.getAmount();
            }
        }

        System.out.println("Expense Report");
        System.out.println("Start Date: " + startDate);
        System.out.println("End Date: " + endDate);
        System.out.println("Total Expenses: " + totalExpenses);
    }
}
