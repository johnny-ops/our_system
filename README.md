import java.time.LocalDate;
import java.time.LocalTime;
import java.util.ArrayList;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        ArrayList<InstructorRecord> records = new ArrayList<>();

        System.out.println("================================================");
        System.out.println("   COLEGIO DE MONTALBAN INSTRUCTOR PAYROLL");
        System.out.println("            MAIN MENU");
        System.out.println("================================================");

        while (true) {
            System.out.println("\n1. Add Employee Record");
            System.out.println("2. View Employee Records");
            System.out.println("3. Exit");
            System.out.print("Select an option (1/2/3): ");
            int option = scanner.nextInt();
            scanner.nextLine(); // Consume newline
            System.out.println("--------------------------------------------------------------------------");
            if (option == 1) {
                addEmployeeRecord(scanner, records);
            } else if (option == 2) {
                viewEmployeeRecords(records);
            } else if (option == 3) {
                System.out.println("\nThank you for using our system!");
                System.out.println("================================================");
                System.out.println("Official Receipt: Payment Received");
                System.out.println("Date: " + LocalDate.now());
                System.out.println("Time: " + LocalTime.now());
                System.out.println("Cashier: Your Name Here");
                System.out.println("================================================");
                break;
            }
        }

        scanner.close();
    }

    private static void addEmployeeRecord(Scanner scanner, ArrayList<InstructorRecord> records) {
        InstructorRecord record = new InstructorRecord();

        // Enter Employee Name with validation for numbers
        String employeeName;
        do {
            System.out.print("Enter Employee Name: ");
            employeeName = scanner.nextLine();
            if (containsNumbers(employeeName)) {
                System.out.println("Error: Employee Name cannot contain numbers. Please try again.");
            }
        } while (containsNumbers(employeeName));
        record.setEmployeeName(employeeName);

        System.out.print("Enter Employee Number: ");
        int employeeNumber = scanner.nextInt();
        record.setEmployeeNumber(employeeNumber);

        System.out.print("Enter Time In (HH:MM): ");
        String timeIn = scanner.next();
        LocalTime loginTime = LocalTime.parse(timeIn);
        record.setLoginTime(loginTime);

        System.out.print("Enter Time Out (HH:MM): ");
        String timeOut = scanner.next();
        LocalTime logoutTime = LocalTime.parse(timeOut);
        record.setLogoutTime(logoutTime);

        // Ask the user for the number of absences
        System.out.print("Enter the number of absences: ");
        int numAbsences = scanner.nextInt();
        record.setNumAbsences(numAbsences);

        // Deduct from the gross salary based on the number of absences
        double grossSalary = calculateGrossSalary(calculateWorkedHours(loginTime, logoutTime));
        double deductionPerAbsence = grossSalary / 30; // Assuming a month has 30 days
        double totalDeduction = numAbsences * deductionPerAbsence;

        record.setGrossSalary(Math.max(0, grossSalary - totalDeduction));

        // Calculate other deductions (SSS, Pag-IBIG, PhilHealth)
        double sssDeduction = calculateSSSDeduction(record.getGrossSalary());
        double pagibigDeduction = calculatePagibigDeduction(record.getGrossSalary());
        double philhealthDeduction = calculatePhilhealthDeduction(record.getGrossSalary());

        record.setSssDeduction(sssDeduction);
        record.setPagibigDeduction(pagibigDeduction);
        record.setPhilhealthDeduction(philhealthDeduction);

        records.add(record);

        // Print a receipt for the added employee record
        double netSalary = calculateNetSalary(record.getGrossSalary(), sssDeduction, pagibigDeduction, philhealthDeduction);

        System.out.println("================================================");
        System.out.println("             PAYROLL RECEIPT            ");
        System.out.println("================================================");
        System.out.println("Employee Name: " + employeeName);
        System.out.println("Employee Number: " + employeeNumber);
        System.out.println("Login Time: " + loginTime);
        System.out.println("Logout Time: " + logoutTime);
        System.out.println("Total Absences (15 Days): " + numAbsences);
        System.out.println("SSS DEDUCTION: ₱" + formatCurrency(sssDeduction));
        System.out.println("PAGIBIG DEDUCTION: ₱" + formatCurrency(pagibigDeduction));
        System.out.println("PHILHEALTH DEDUCTION: ₱" + formatCurrency(philhealthDeduction));
        System.out.println("Hours Worked: " + calculateWorkedHours(loginTime, logoutTime) + " hours");
        System.out.println("Hourly Rate: ₱500 per hour");
        System.out.println("Gross Salary: ₱" + formatCurrency(record.getGrossSalary()));
        System.out.println("Total Deductions: ₱" + formatCurrency(grossSalary - netSalary));
        System.out.println("Net Salary: ₱" + formatCurrency(netSalary));
        System.out.println("================================================");
        System.out.println("Employee record added successfully.");
    }

    private static void viewEmployeeRecords(ArrayList<InstructorRecord> records) {
        if (records.isEmpty()) {
            System.out.println("No employee records to display.");
        } else {
            System.out.println("Instructor Records:");
            System.out.println("------------------------------------------------------------------------------------------------------------------------------------------------------");
            System.out.printf("| %-20s | %-15s | %-18s | %-15s | %-15s | %-20s | %-18s |%n",
                    "Employee Name", "Employee Number", "Gross Salary (PHP)", "Net Salary (PHP)", "Worked Hours", "Total Absences (15 Days)", "Total Salary (15 Days)");
            System.out.println("------------------------------------------------------------------------------------------------------------------------------------------------------");

            calculateTotalSalaryForPast15Days(records);

            for (InstructorRecord r : records) {
                double workedHours = calculateWorkedHours(r.getLoginTime(), r.getLogoutTime());
                double grossSalary = calculateGrossSalary(workedHours);
                double netSalary = calculateNetSalary(grossSalary, r.getSssDeduction(), r.getPagibigDeduction(), r.getPhilhealthDeduction());

                System.out.printf("| %-20s | %-15s | ₱%-16.2f | ₱%-15.2f | %.2f hours       |      %-18d      |     ₱%-15.2f    |%n",
                        r.getEmployeeName(), r.getEmployeeNumber(), grossSalary, netSalary, workedHours, r.getNumAbsences(), r.getTotalSalaryForPast15Days());
            }
            System.out.println("-------------------------------------------------------------------------------------------------------------------------------------------------------");
        }
    }

    private static void calculateTotalSalaryForPast15Days(ArrayList<InstructorRecord> records) {
        LocalDate currentDate = LocalDate.now();
        LocalDate startDate = currentDate.minusDays(15);

        for (InstructorRecord record : records) {
            LocalDate loginDate = currentDate.minusDays(1);
            if (record.getLoginTime() != null) {
                loginDate = loginDate.minusDays(1); // Assuming the login date is the day before the current date.
            }

            if (!loginDate.isBefore(startDate)) {
                double totalSalary = 0.0;
                double totalDeduction = 0.0;
                LocalDate recordDate = loginDate;

                while (!recordDate.isAfter(currentDate)) {
                    double workedHours = calculateWorkedHours(record.getLoginTime(), record.getLogoutTime());
                    double grossSalary = calculateGrossSalary(workedHours);
                    double netSalary = calculateNetSalary(grossSalary, record.getSssDeduction(), record.getPagibigDeduction(), record.getPhilhealthDeduction());

                    totalSalary += grossSalary;
                    totalDeduction += (grossSalary - netSalary);
                    recordDate = recordDate.plusDays(1);
                }

                record.setTotalSalaryForPast15Days(totalSalary - totalDeduction);
            }
        }
    }

    private static double calculateGrossSalary(double workedHours) {
        // Assuming ₱10 hourly rate
        return workedHours * 500;
    }

    private static double calculateNetSalary(double grossSalary, double sssDeduction, double pagibigDeduction, double philhealthDeduction) {
        return grossSalary - (sssDeduction + pagibigDeduction + philhealthDeduction);
    }

    private static double calculateWorkedHours(LocalTime loginTime, LocalTime logoutTime) {
        int hours = logoutTime.getHour() - loginTime.getHour();
        int minutes = logoutTime.getMinute() - loginTime.getMinute();
        return hours + (minutes / 60.0);
    }

    private static double calculateSSSDeduction(double grossSalary) {
        if (grossSalary <= 10000) {
            return grossSalary * 0.10; // 10% deduction
        }
        return (grossSalary > 20000) ? 800.00 : 0.00;
    }

    private static double calculatePagibigDeduction(double grossSalary) {
        if (grossSalary <= 10000) {
            return grossSalary * 0.20; // 20% deduction
        }
        return (grossSalary > 20000) ? 100.00 : 0.00;
    }

    private static double calculatePhilhealthDeduction(double grossSalary) {
        if (grossSalary <= 10000) {
            return grossSalary * 0.25; // 25% deduction
        }
        return (grossSalary > 20000) ? 300.00 : 0.00;
    }

    private static String formatCurrency(double amount) {
        // Format currency without negative sign and with 2 decimal places
        return String.format("₱%.2f", Math.max(0, amount));
    }

    private static boolean containsNumbers(String input) {
        // Check if the string contains any numerical digits
        return input.matches(".*\\d.*");
    }
}

class InstructorRecord {
    private String employeeName;
    private int employeeNumber;
    private LocalTime loginTime;
    private LocalTime logoutTime;
    private int numAbsences; // New field to represent the number of absences
    private double grossSalary;
    private double sssDeduction;
    private double pagibigDeduction;
    private double philhealthDeduction;
    private double totalSalaryForPast15Days;

    public void setEmployeeName(String name) {
        employeeName = name;
    }

    public void setEmployeeNumber(int number) {
        employeeNumber = number;
    }

    public void setLoginTime(LocalTime time) {
        loginTime = time;
    }

    public void setLogoutTime(LocalTime time) {
        logoutTime = time;
    }

    public void setNumAbsences(int absences) {
        numAbsences = absences;
    }

    public void setGrossSalary(double salary) {
        grossSalary = salary;
    }

    public void setSssDeduction(double sss) {
        sssDeduction = sss;
    }

    public void setPagibigDeduction(double pagibig) {
        pagibigDeduction = pagibig;
    }

    public void setPhilhealthDeduction(double philhealth) {
        philhealthDeduction = philhealth;
    }

    public void setTotalSalaryForPast15Days(double totalSalary) {
        totalSalaryForPast15Days = totalSalary;
    }

    public String getEmployeeName() {
        return employeeName;
    }

    public int getEmployeeNumber() {
        return employeeNumber;
    }

    public LocalTime getLoginTime() {
        return loginTime;
    }

    public LocalTime getLogoutTime() {
        return logoutTime;
    }

    public int getNumAbsences() {
        return numAbsences;
    }

    public double getGrossSalary() {
        return grossSalary;
    }

    public double getSssDeduction() {
        return sssDeduction;
    }

    public double getPagibigDeduction() {
        return pagibigDeduction;
    }

    public double getPhilhealthDeduction() {
        return philhealthDeduction;
    }

    public double getTotalSalaryForPast15Days() {
        return totalSalaryForPast15Days;
    }
}
