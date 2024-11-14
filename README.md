import javax.swing.*;
import java.awt.*;
import java.util.*;
import java.text.SimpleDateFormat;

class MonCanevas extends JPanel {
    public void paint(Graphics g) {
        super.paint(g);
        g.setColor(Color.red);
        g.drawString("Hello World!", 100, 150);
        g.drawLine(100, 160, 170, 160);
    }
}

class Employee {
    private int id;
    private String name;
    private String surname;
    private java.util.List<String> checkIn;
    private java.util.List<String> checkOut;
    private double wage;
    private YearlyStats yearlyStats;

    public Employee(int id, String name, String surname, double wage) {
        this.id = id;
        this.name = name;
        this.surname = surname;
        this.wage = wage;
        this.checkIn = new ArrayList<>();
        this.checkOut = new ArrayList<>();
        this.yearlyStats = new YearlyStats();
    }

    public void addCheckIn(String time) { checkIn.add(time); }
    public void addCheckOut(String time) { checkOut.add(time); }
    public java.util.List<String> getCheckIn() { return checkIn; }
    public java.util.List<String> getCheckOut() { return checkOut; }
    public double getWage() { return wage; }
    public String getFullName() { return name + " " + surname; }
    public YearlyStats getYearlyStats() { return yearlyStats; }
}

class YearlyStats {
    private double totalDeductions;
    private double totalBonuses;
    private double totalWages;

    public void addWages(double amount) { totalWages += amount; }
    public void addDeductions(double amount) { totalDeductions += amount; }
    public void addBonuses(double amount) { totalBonuses += amount; }
    public double getTotalDeductions() { return totalDeductions; }
    public double getTotalBonuses() { return totalBonuses; }
    public double getTotalWages() { return totalWages; }
}

class EmployeeService {
    private static final double earlyArrivalBonus = 2000;
    private static final double lateDeductionRate = 0.001;
    private static final String workStartTime = "08:00";
    private java.util.List<Employee> employees;

    public EmployeeService() {
        employees = new ArrayList<>();
        employees.add(new Employee(1, "Said", "Benhadj", 50000));
        employees.add(new Employee(2, "Amel", "Kouider", 48000));
        employees.add(new Employee(3, "Ali", "Mekhloufi", 52000));
        employees.add(new Employee(4, "Fatima", "Boualem", 47000));
    }

    public void simulateCheckInOut() {
        employees.forEach(employee -> {
            employee.addCheckIn(generateRandomTime("08:00", 30));
            employee.addCheckOut(generateRandomTime("16:00", 60));
        });
    }

    public String calculateWages(String selectedPackage) {
        double packageModifier = getPackageModifier(selectedPackage);
        StringBuilder report = new StringBuilder();

        for (Employee employee : employees) {
            double totalDeduction = calculateDeductions(employee.getCheckIn(), employee.getWage());
            double netWage = employee.getWage() + packageModifier - totalDeduction;
            boolean earlyArrival = isEarlyArrival(employee.getCheckIn());

            if (earlyArrival) {
                netWage += earlyArrivalBonus;
                employee.getYearlyStats().addBonuses(earlyArrivalBonus);
            }

            employee.getYearlyStats().addWages(netWage);
            employee.getYearlyStats().addDeductions(totalDeduction);

            report.append(displayEmployeeReport(employee, netWage, totalDeduction, earlyArrival, packageModifier));
        }
        return report.toString();
    }

    private double getPackageModifier(String selectedPackage) {
        switch (selectedPackage.toLowerCase()) {
            case "premium": return 5000;
            case "deluxe": return 10000;
            default: return 0;
        }
    }

    private double calculateDeductions(java.util.List<String> checkInTimes, double wage) {
        double totalDeduction = 0;
        for (String checkIn : checkInTimes) {
            if (!isValidCheckIn(checkIn)) {
                int delay = getMinutesDifference(workStartTime, checkIn);
                totalDeduction += wage * lateDeductionRate * delay;
            }
        }
        return totalDeduction;
    }

    private boolean isValidCheckIn(String checkInTime) {
        return getMinutesDifference(workStartTime, checkInTime) <= 0;
    }

    private int getMinutesDifference(String startTime, String time) {
        try {
            SimpleDateFormat format = new SimpleDateFormat("HH:mm");
            Date startDate = format.parse(startTime);
            Date checkInDate = format.parse(time);
            return (int) ((checkInDate.getTime() - startDate.getTime()) / (60 * 1000));
        } catch (Exception e) {
            return 0;
        }
    }

    private boolean isEarlyArrival(java.util.List<String> checkInTimes) {
        for (String checkIn : checkInTimes) {
            if (isValidCheckIn(checkIn)) {
                return true;
            }
        }
        return false;
    }

    private String generateRandomTime(String baseTime, int maxOffset) {
        try {
            SimpleDateFormat format = new SimpleDateFormat("HH:mm");
            Date baseDate = format.parse(baseTime);
            Random random = new Random();
            int offsetMinutes = random.nextInt(maxOffset * 2 + 1) - maxOffset;
            return format.format(new Date(baseDate.getTime() + offsetMinutes * 60 * 1000));
        } catch (Exception e) {
            return baseTime;
        }
    }

    private String displayEmployeeReport(Employee employee, double netWage, double totalDeduction, boolean earlyArrival, double packageModifier) {
        StringBuilder report = new StringBuilder();
        report.append("Employee: ").append(employee.getFullName()).append("\n");
        report.append("Wage: ").append(employee.getWage()).append(" DZD\n");
        report.append("Package Modifier: ").append(packageModifier).append(" DZD\n");
        report.append("Total Deduction: ").append(totalDeduction).append(" DZD\n");
        if (earlyArrival) report.append("Bonus: ").append(earlyArrivalBonus).append(" DZD\n");
        report.append("Net Wage: ").append(netWage).append(" DZD\n");
        return report.toString();
    }

    public void consoleReport(String packageType) {
        System.out.println("Employee Wage Report:");
        System.out.println(calculateWages(packageType));
    }
}

class MainFrame extends JFrame {
    private JTextField companyNameField, passwordField, packageField;
    private JTextArea outputArea;
    private EmployeeService employeeService;

    public MainFrame() {
        setTitle("Employee Wage System with Drawing");
        setSize(600, 400);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new FlowLayout());

        employeeService = new EmployeeService();

        add(new JLabel("Company Name:"));
        companyNameField = new JTextField(15);
        add(companyNameField);

        add(new JLabel("Password:"));
        passwordField = new JTextField(15);
        add(passwordField);

        JButton loginButton = new JButton("Login");
        add(loginButton);

        add(new JLabel("Package (standard, premium, deluxe):"));
        packageField = new JTextField(10);
        add(packageField);

        JButton calculateButton = new JButton("Calculate Wages");
        add(calculateButton);

        outputArea = new JTextArea(10, 50);
        outputArea.setEditable(false);
        add(new JScrollPane(outputArea));

        loginButton.addActionListener(e -> login());
        calculateButton.addActionListener(e -> calculateWages());

        MonCanevas monCanevas = new MonCanevas();
        monCanevas.setPreferredSize(new Dimension(290, 330));
        add(monCanevas);
    }

    private void login() {
        outputArea.setText("Registration Successful!\n");
        employeeService.simulateCheckInOut();
    }

    private void calculateWages() {
        String packageType = packageField.getText();
        String report = employeeService.calculateWages(packageType);
        outputArea.append(report);
    }
}

public class Main {
    public static void main(String[] args) {
        if (GraphicsEnvironment.isHeadless()) {
            EmployeeService employeeService = new EmployeeService();
            employeeService.simulateCheckInOut();
            employeeService.consoleReport("standard");
        } else {
            SwingUtilities.invokeLater(() -> {
                MainFrame frame = new MainFrame();
                frame.setVisible(true);
            });
        }
    }
}
