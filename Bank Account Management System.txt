import java.io.*;
import java.util.*;
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import javax.swing.border.EmptyBorder;

class Account {
    private int accountNumber;
    private String name;
    private String phoneNumber;
    private String dob;
    private float balance;
    private float previousBalance;
    private boolean isActive;

    public Account(int accountNumber, String name, String phoneNumber, String dob, float balance) {
        this.accountNumber = accountNumber;
        this.name = name;
        this.phoneNumber = phoneNumber;
        this.dob = dob;
        this.balance = balance;
        this.previousBalance = 0.0f;
        this.isActive = true;
    }

    // (Encapsulation)
    public int getAccountNumber() { return accountNumber; }
    public String getName() { return name; }
    public String getPhoneNumber() { return phoneNumber; }
    public String getDob() { return dob; }
    public float getBalance() { return balance; }
    public float getPreviousBalance() { return previousBalance; }
    public boolean isActive() { return isActive; }

    public void setPhoneNumber(String phoneNumber) { this.phoneNumber = phoneNumber; }
    public void setActive(boolean active) { isActive = active; }

    public void deposit(float amount) {
        previousBalance = balance;
        balance += amount;
    }

    public boolean withdraw(float amount) {
        if (amount > balance) return false;
        previousBalance = balance;
        balance -= amount;
        return true;
    }

    public boolean transferTo(Account receiver, float amount) {
        if (withdraw(amount)) {
            receiver.deposit(amount);
            return true;
        }
        return false;
    }

    @Override
    public String toString() {
        return accountNumber + " " + name + " " + phoneNumber + " " + dob + " " +
                balance + " " + previousBalance + " " + (isActive ? 1 : 0);
    }

    public static Account fromString(String data) {
        String[] parts = data.split(" ");
        Account account = new Account(
                Integer.parseInt(parts[0]),
                parts[1],
                parts[2],
                parts[3],
                Float.parseFloat(parts[4])
        );
        account.previousBalance = Float.parseFloat(parts[5]);
        account.setActive(parts[6].equals("1"));
        return account;
    }
}

public class BankSystem extends JFrame {
    private static final String FILE_PATH = "accounts.txt";
    // Use fully qualified name for ArrayList to avoid ambiguity
    private final java.util.List<Account> accounts = new java.util.ArrayList<>();
    private JPanel mainPanel;
    private CardLayout cardLayout;

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
            } catch (Exception e) {
                e.printStackTrace();
            }
            new BankSystem().setVisible(true);
        });
    }

    public BankSystem() {
        setTitle("Bank Account Management System");
        setSize(600, 400);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        loadData();
        initializeUI();
    }

    private void initializeUI() {
        cardLayout = new CardLayout();
        mainPanel = new JPanel(cardLayout);

        // Create main menu panel
        JPanel menuPanel = createMenuPanel();
        mainPanel.add(menuPanel, "MENU");

        // Create panels for each feature
        mainPanel.add(createAccountPanel(), "CREATE");
        mainPanel.add(updateAccountPanel(), "UPDATE");
        mainPanel.add(closeAccountPanel(), "CLOSE");
        mainPanel.add(depositPanel(), "DEPOSIT");
        mainPanel.add(withdrawPanel(), "WITHDRAW");
        mainPanel.add(transferPanel(), "TRANSFER");
        mainPanel.add(viewDetailsPanel(), "VIEW");
        mainPanel.add(totalMoneyPanel(), "TOTAL");

        add(mainPanel);
        cardLayout.show(mainPanel, "MENU");

        // Add window listener to save data when closing
        addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                saveData();
            }
        });
    }

    private JPanel createMenuPanel() {
        JPanel panel = new JPanel(new BorderLayout());

        // Title panel
        JPanel titlePanel = new JPanel();
        JLabel titleLabel = new JLabel("Bank Account Management System");
        titleLabel.setFont(new Font("Arial", Font.BOLD, 20));
        titlePanel.add(titleLabel);
        panel.add(titlePanel, BorderLayout.NORTH);

        // Button panel
        JPanel buttonPanel = new JPanel(new GridLayout(4, 2, 10, 10));
        buttonPanel.setBorder(new EmptyBorder(20, 50, 20, 50));

        String[] buttonLabels = {
                "Create Account", "Update Account", "Close Account", "Deposit Funds",
                "Withdraw Funds", "Transfer Funds", "View Account Details", "View Total Bank Money"
        };

        String[] destinations = {
                "CREATE", "UPDATE", "CLOSE", "DEPOSIT",
                "WITHDRAW", "TRANSFER", "VIEW", "TOTAL"
        };

        for (int i = 0; i < buttonLabels.length; i++) {
            JButton button = new JButton(buttonLabels[i]);
            final String dest = destinations[i];
            button.addActionListener(e -> cardLayout.show(mainPanel, dest));
            buttonPanel.add(button);
        }

        panel.add(buttonPanel, BorderLayout.CENTER);

        // Exit button
        JPanel bottomPanel = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        JButton exitButton = new JButton("Exit");
        exitButton.addActionListener(e -> {
            saveData();
            System.exit(0);
        });
        bottomPanel.add(exitButton);
        panel.add(bottomPanel, BorderLayout.SOUTH);

        return panel;
    }

    private JPanel createAccountPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBorder(new EmptyBorder(20, 20, 20, 20));

        JPanel formPanel = new JPanel(new GridLayout(6, 2, 10, 10));

        JLabel accNumberLabel = new JLabel("Account Number:");
        JTextField accNumberField = new JTextField();

        JLabel nameLabel = new JLabel("Name:");
        JTextField nameField = new JTextField();

        JLabel phoneLabel = new JLabel("Phone Number:");
        JTextField phoneField = new JTextField();

        JLabel dobLabel = new JLabel("Date of Birth (DD-MM-YYYY):");
        JTextField dobField = new JTextField();

        JLabel balanceLabel = new JLabel("Initial Balance:");
        JTextField balanceField = new JTextField();

        formPanel.add(accNumberLabel);
        formPanel.add(accNumberField);
        formPanel.add(nameLabel);
        formPanel.add(nameField);
        formPanel.add(phoneLabel);
        formPanel.add(phoneField);
        formPanel.add(dobLabel);
        formPanel.add(dobField);
        formPanel.add(balanceLabel);
        formPanel.add(balanceField);

        JButton createButton = new JButton("Create Account");
        JButton backButton = new JButton("Back to Menu");

        createButton.addActionListener(e -> {
            try {
                int num = Integer.parseInt(accNumberField.getText());

                if (getAccount(num) != null) {
                    JOptionPane.showMessageDialog(this, "Account number already exists.", "Error", JOptionPane.ERROR_MESSAGE);
                    return;
                }

                String name = nameField.getText();
                String phone = phoneField.getText();
                String dob = dobField.getText();
                float balance = Float.parseFloat(balanceField.getText());

                accounts.add(new Account(num, name, phone, dob, balance));
                JOptionPane.showMessageDialog(this, "Account created successfully.", "Success", JOptionPane.INFORMATION_MESSAGE);

                // Clear fields
                accNumberField.setText("");
                nameField.setText("");
                phoneField.setText("");
                dobField.setText("");
                balanceField.setText("");
                saveData();
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Please enter valid numbers.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        });

        backButton.addActionListener(e -> cardLayout.show(mainPanel, "MENU"));

        JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        buttonPanel.add(createButton);
        buttonPanel.add(backButton);

        panel.add(new JLabel("Create New Account", JLabel.CENTER), BorderLayout.NORTH);
        panel.add(formPanel, BorderLayout.CENTER);
        panel.add(buttonPanel, BorderLayout.SOUTH);

        return panel;
    }

    private JPanel updateAccountPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBorder(new EmptyBorder(20, 20, 20, 20));

        JPanel formPanel = new JPanel(new GridLayout(2, 2, 10, 10));

        JLabel accNumberLabel = new JLabel("Account Number:");
        JTextField accNumberField = new JTextField();

        JLabel phoneLabel = new JLabel("New Phone Number:");
        JTextField phoneField = new JTextField();

        formPanel.add(accNumberLabel);
        formPanel.add(accNumberField);
        formPanel.add(phoneLabel);
        formPanel.add(phoneField);

        JButton updateButton = new JButton("Update Account");
        JButton backButton = new JButton("Back to Menu");

        updateButton.addActionListener(e -> {
            try {
                int accNum = Integer.parseInt(accNumberField.getText());
                Account acc = getAccount(accNum);

                if (acc != null) {
                    acc.setPhoneNumber(phoneField.getText());
                    JOptionPane.showMessageDialog(this, "Phone number updated successfully.", "Success", JOptionPane.INFORMATION_MESSAGE);

                    // Clear fields
                    accNumberField.setText("");
                    phoneField.setText("");
                    saveData();
                } else {
                    JOptionPane.showMessageDialog(this, "Account not found.", "Error", JOptionPane.ERROR_MESSAGE);
                }
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Please enter valid account number.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        });

        backButton.addActionListener(e -> cardLayout.show(mainPanel, "MENU"));

        JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        buttonPanel.add(updateButton);
        buttonPanel.add(backButton);

        panel.add(new JLabel("Update Account", JLabel.CENTER), BorderLayout.NORTH);
        panel.add(formPanel, BorderLayout.CENTER);
        panel.add(buttonPanel, BorderLayout.SOUTH);

        return panel;
    }

    private JPanel closeAccountPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBorder(new EmptyBorder(20, 20, 20, 20));

        JPanel formPanel = new JPanel(new GridLayout(1, 2, 10, 10));

        JLabel accNumberLabel = new JLabel("Account Number:");
        JTextField accNumberField = new JTextField();

        formPanel.add(accNumberLabel);
        formPanel.add(accNumberField);

        JButton closeButton = new JButton("Close Account");
        JButton backButton = new JButton("Back to Menu");

        closeButton.addActionListener(e -> {
            try {
                int accNum = Integer.parseInt(accNumberField.getText());
                Account acc = getAccount(accNum);

                if (acc != null) {
                    acc.setActive(false);
                    JOptionPane.showMessageDialog(this, "Account closed successfully.", "Success", JOptionPane.INFORMATION_MESSAGE);

                    // Clear field
                    accNumberField.setText("");
                    saveData();
                } else {
                    JOptionPane.showMessageDialog(this, "Account not found.", "Error", JOptionPane.ERROR_MESSAGE);
                }
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Please enter valid account number.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        });

        backButton.addActionListener(e -> cardLayout.show(mainPanel, "MENU"));

        JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        buttonPanel.add(closeButton);
        buttonPanel.add(backButton);

        panel.add(new JLabel("Close Account", JLabel.CENTER), BorderLayout.NORTH);
        panel.add(formPanel, BorderLayout.CENTER);
        panel.add(buttonPanel, BorderLayout.SOUTH);

        return panel;
    }

    private JPanel depositPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBorder(new EmptyBorder(20, 20, 20, 20));

        JPanel formPanel = new JPanel(new GridLayout(2, 2, 10, 10));

        JLabel accNumberLabel = new JLabel("Account Number:");
        JTextField accNumberField = new JTextField();

        JLabel amountLabel = new JLabel("Amount to Deposit:");
        JTextField amountField = new JTextField();

        formPanel.add(accNumberLabel);
        formPanel.add(accNumberField);
        formPanel.add(amountLabel);
        formPanel.add(amountField);

        JButton depositButton = new JButton("Deposit");
        JButton backButton = new JButton("Back to Menu");

        depositButton.addActionListener(e -> {
            try {
                int accNum = Integer.parseInt(accNumberField.getText());
                float amount = Float.parseFloat(amountField.getText());
                Account acc = getAccount(accNum);

                if (acc != null && acc.isActive()) {
                    acc.deposit(amount);
                    JOptionPane.showMessageDialog(this,
                            String.format("Deposited. New Balance: %.2f", acc.getBalance()),
                            "Success", JOptionPane.INFORMATION_MESSAGE);

                    // Clear fields
                    accNumberField.setText("");
                    amountField.setText("");
                    saveData();
                } else {
                    JOptionPane.showMessageDialog(this, "Account not found or inactive.", "Error", JOptionPane.ERROR_MESSAGE);
                }
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Please enter valid numbers.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        });

        backButton.addActionListener(e -> cardLayout.show(mainPanel, "MENU"));

        JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        buttonPanel.add(depositButton);
        buttonPanel.add(backButton);

        panel.add(new JLabel("Deposit Funds", JLabel.CENTER), BorderLayout.NORTH);
        panel.add(formPanel, BorderLayout.CENTER);
        panel.add(buttonPanel, BorderLayout.SOUTH);

        return panel;
    }

    private JPanel withdrawPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBorder(new EmptyBorder(20, 20, 20, 20));

        JPanel formPanel = new JPanel(new GridLayout(2, 2, 10, 10));

        JLabel accNumberLabel = new JLabel("Account Number:");
        JTextField accNumberField = new JTextField();

        JLabel amountLabel = new JLabel("Amount to Withdraw:");
        JTextField amountField = new JTextField();

        formPanel.add(accNumberLabel);
        formPanel.add(accNumberField);
        formPanel.add(amountLabel);
        formPanel.add(amountField);

        JButton withdrawButton = new JButton("Withdraw");
        JButton backButton = new JButton("Back to Menu");

        withdrawButton.addActionListener(e -> {
            try {
                int accNum = Integer.parseInt(accNumberField.getText());
                float amount = Float.parseFloat(amountField.getText());
                Account acc = getAccount(accNum);

                if (acc != null && acc.isActive()) {
                    if (acc.withdraw(amount)) {
                        JOptionPane.showMessageDialog(this,
                                String.format("Withdrawn. New Balance: %.2f", acc.getBalance()),
                                "Success", JOptionPane.INFORMATION_MESSAGE);

                        // Clear fields
                        accNumberField.setText("");
                        amountField.setText("");
                        saveData();
                    } else {
                        JOptionPane.showMessageDialog(this, "Insufficient balance.", "Error", JOptionPane.ERROR_MESSAGE);
                    }
                } else {
                    JOptionPane.showMessageDialog(this, "Account not found or inactive.", "Error", JOptionPane.ERROR_MESSAGE);
                }
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Please enter valid numbers.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        });

        backButton.addActionListener(e -> cardLayout.show(mainPanel, "MENU"));

        JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        buttonPanel.add(withdrawButton);
        buttonPanel.add(backButton);

        panel.add(new JLabel("Withdraw Funds", JLabel.CENTER), BorderLayout.NORTH);
        panel.add(formPanel, BorderLayout.CENTER);
        panel.add(buttonPanel, BorderLayout.SOUTH);

        return panel;
    }

    private JPanel transferPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBorder(new EmptyBorder(20, 20, 20, 20));

        JPanel formPanel = new JPanel(new GridLayout(3, 2, 10, 10));

        JLabel fromLabel = new JLabel("From Account Number:");
        JTextField fromField = new JTextField();

        JLabel toLabel = new JLabel("To Account Number:");
        JTextField toField = new JTextField();

        JLabel amountLabel = new JLabel("Amount to Transfer:");
        JTextField amountField = new JTextField();

        formPanel.add(fromLabel);
        formPanel.add(fromField);
        formPanel.add(toLabel);
        formPanel.add(toField);
        formPanel.add(amountLabel);
        formPanel.add(amountField);

        JButton transferButton = new JButton("Transfer");
        JButton backButton = new JButton("Back to Menu");

        transferButton.addActionListener(e -> {
            try {
                int fromAcc = Integer.parseInt(fromField.getText());
                int toAcc = Integer.parseInt(toField.getText());
                float amount = Float.parseFloat(amountField.getText());

                Account sender = getAccount(fromAcc);
                Account receiver = getAccount(toAcc);

                if (sender == null || !sender.isActive()) {
                    JOptionPane.showMessageDialog(this, "Sender account invalid.", "Error", JOptionPane.ERROR_MESSAGE);
                    return;
                }

                if (receiver == null || !receiver.isActive()) {
                    JOptionPane.showMessageDialog(this, "Receiver account invalid.", "Error", JOptionPane.ERROR_MESSAGE);
                    return;
                }

                if (sender.transferTo(receiver, amount)) {
                    JOptionPane.showMessageDialog(this, "Transfer successful.", "Success", JOptionPane.INFORMATION_MESSAGE);

                    // Clear fields
                    fromField.setText("");
                    toField.setText("");
                    amountField.setText("");
                    saveData();
                } else {
                    JOptionPane.showMessageDialog(this, "Transfer failed due to insufficient balance.", "Error", JOptionPane.ERROR_MESSAGE);
                }
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Please enter valid numbers.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        });

        backButton.addActionListener(e -> cardLayout.show(mainPanel, "MENU"));

        JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        buttonPanel.add(transferButton);
        buttonPanel.add(backButton);

        panel.add(new JLabel("Transfer Funds", JLabel.CENTER), BorderLayout.NORTH);
        panel.add(formPanel, BorderLayout.CENTER);
        panel.add(buttonPanel, BorderLayout.SOUTH);

        return panel;
    }

    private JPanel viewDetailsPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBorder(new EmptyBorder(20, 20, 20, 20));

        JPanel topPanel = new JPanel(new FlowLayout(FlowLayout.CENTER));

        JLabel accNumberLabel = new JLabel("Account Number:");
        JTextField accNumberField = new JTextField(10);
        JButton viewButton = new JButton("View");

        topPanel.add(accNumberLabel);
        topPanel.add(accNumberField);
        topPanel.add(viewButton);

        JTextArea detailsArea = new JTextArea(10, 30);
        detailsArea.setEditable(false);
        JScrollPane scrollPane = new JScrollPane(detailsArea);

        JPanel centerPanel = new JPanel(new BorderLayout());
        centerPanel.add(topPanel, BorderLayout.NORTH);
        centerPanel.add(scrollPane, BorderLayout.CENTER);

        viewButton.addActionListener(e -> {
            try {
                int accNum = Integer.parseInt(accNumberField.getText());
                Account acc = getAccount(accNum);

                if (acc != null) {
                    StringBuilder details = new StringBuilder();
                    details.append("Account Number: ").append(acc.getAccountNumber()).append("\n");
                    details.append("Name: ").append(acc.getName()).append("\n");
                    details.append("Phone: ").append(acc.getPhoneNumber()).append("\n");
                    details.append("DOB: ").append(acc.getDob()).append("\n");
                    details.append(String.format("Current Balance: %.2f\n", acc.getBalance()));
                    details.append(String.format("Previous Balance: %.2f\n", acc.getPreviousBalance()));
                    details.append("Status: ").append(acc.isActive() ? "Active" : "Closed");

                    detailsArea.setText(details.toString());
                } else {
                    detailsArea.setText("Account not found.");
                }
            } catch (NumberFormatException ex) {
                detailsArea.setText("Please enter valid account number.");
            }
        });

        JButton backButton = new JButton("Back to Menu");
        backButton.addActionListener(e -> cardLayout.show(mainPanel, "MENU"));

        JPanel bottomPanel = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        bottomPanel.add(backButton);

        panel.add(new JLabel("View Account Details", JLabel.CENTER), BorderLayout.NORTH);
        panel.add(centerPanel, BorderLayout.CENTER);
        panel.add(bottomPanel, BorderLayout.SOUTH);

        return panel;
    }

    private JPanel totalMoneyPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBorder(new EmptyBorder(20, 20, 20, 20));

        JButton calculateButton = new JButton("Calculate Total Money");
        JLabel resultLabel = new JLabel("Total Bank Money: 0.00");
        resultLabel.setFont(new Font("Arial", Font.BOLD, 16));

        calculateButton.addActionListener(e -> {
            float total = 0;
            for (Account acc : accounts) {
                if (acc.isActive()) total += acc.getBalance();
            }
            resultLabel.setText(String.format("Total Bank Money: %.2f", total));
        });

        JButton backButton = new JButton("Back to Menu");
        backButton.addActionListener(e -> cardLayout.show(mainPanel, "MENU"));

        JPanel centerPanel = new JPanel();
        centerPanel.setLayout(new BoxLayout(centerPanel, BoxLayout.Y_AXIS));

        calculateButton.setAlignmentX(Component.CENTER_ALIGNMENT);
        resultLabel.setAlignmentX(Component.CENTER_ALIGNMENT);

        centerPanel.add(Box.createVerticalGlue());
        centerPanel.add(calculateButton);
        centerPanel.add(Box.createRigidArea(new Dimension(0, 20)));
        centerPanel.add(resultLabel);
        centerPanel.add(Box.createVerticalGlue());

        JPanel bottomPanel = new JPanel(new FlowLayout(FlowLayout.RIGHT));
        bottomPanel.add(backButton);

        panel.add(new JLabel("View Total Bank Money", JLabel.CENTER), BorderLayout.NORTH);
        panel.add(centerPanel, BorderLayout.CENTER);
        panel.add(bottomPanel, BorderLayout.SOUTH);

        return panel;
    }

    private Account getAccount(int accNum) {
        for (Account acc : accounts) {
            if (acc.getAccountNumber() == accNum) {
                return acc;
            }
        }
        return null;
    }

    private void saveData() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(FILE_PATH))) {
            for (Account acc : accounts) {
                writer.write(acc.toString());
                writer.newLine();
            }
        } catch (IOException e) {
            JOptionPane.showMessageDialog(this, "Error saving data.", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void loadData() {
        File file = new File(FILE_PATH);
        if (!file.exists()) {
            System.out.println("No previous data found.");
            return;
        }

        try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = reader.readLine()) != null) {
                accounts.add(Account.fromString(line));
            }
        } catch (IOException e) {
            JOptionPane.showMessageDialog(this, "Error loading data.", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }
}
