# dental-clinic-java-jdbc
//setup myeclipse or any ide compatible, then connect using jdbc driver to mysql and the project folder.
//save this code in single file same directory  where the driver is also set up. can use multiple packages also but not needed.
for desktop usage using java frame,and backend mysql.
//java jframes used for front end and mysql for backend

     
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.sql.*;
import java.util.ArrayList;

public class DentalClinicApp extends JFrame {
    // Database Configuration
    private static final String DB_URL = "jdbc:mysql://localhost:3306/dental_clinic";
    private static final String DB_USER = "root";
    private static final String DB_PASSWORD = "gangsterafaghaliya";

    private Connection conn;

    public DentalClinicApp() {
        // Setup GUI
        setTitle("Dental Clinic Management");
        setSize(600, 400);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLayout(new FlowLayout());

        JButton addPatientButton = new JButton("Add Patient");
        JButton bookAppointmentButton = new JButton("Book Appointment");
        JButton viewAppointmentsButton = new JButton("View Appointments");
        JButton updateAppointmentButton = new JButton("Update Appointment");
        JButton deleteAppointmentButton = new JButton("Delete Appointment");
        JButton deletePatientButton = new JButton("Delete Patient");

        add(addPatientButton);
        add(bookAppointmentButton);
        add(viewAppointmentsButton);
        add(updateAppointmentButton);
        add(deleteAppointmentButton);
        add(deletePatientButton);

        addPatientButton.addActionListener(this::showAddPatientForm);
        bookAppointmentButton.addActionListener(this::showBookAppointmentForm);
        viewAppointmentsButton.addActionListener(this::viewAppointmentsAction);
        updateAppointmentButton.addActionListener(this::updateAppointmentAction);
        deleteAppointmentButton.addActionListener(this::deleteAppointmentAction);
        deletePatientButton.addActionListener(this::deletePatientAction);

        // Establish database connection
        try {
            conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
        } catch (SQLException e) {
            JOptionPane.showMessageDialog(this, "Database connection failed: " + e.getMessage());
            System.exit(1);
        }
    }

    // Add Patient Form
    private void showAddPatientForm(ActionEvent e) {
        JPanel panel = new JPanel(new GridLayout(4, 2, 10, 10));
        JTextField nameField = new JTextField();
        JTextField ageField = new JTextField();
        JTextField genderField = new JTextField();
        JTextField contactField = new JTextField();

        panel.add(new JLabel("Name:"));
        panel.add(nameField);
        panel.add(new JLabel("Age:"));
        panel.add(ageField);
        panel.add(new JLabel("Gender (Male/Female):"));
        panel.add(genderField);
        panel.add(new JLabel("Contact:"));
        panel.add(contactField);

        int result = JOptionPane.showConfirmDialog(this, panel, "Add Patient", JOptionPane.OK_CANCEL_OPTION);
        if (result == JOptionPane.OK_OPTION) {
            try {
                String name = nameField.getText();
                int age = Integer.parseInt(ageField.getText());
                String gender = genderField.getText();
                String contact = contactField.getText();

                String sql = "INSERT INTO patients (name, age, gender, contact) VALUES (?, ?, ?, ?)";
                try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
                    pstmt.setString(1, name);
                    pstmt.setInt(2, age);
                    pstmt.setString(3, gender);
                    pstmt.setString(4, contact);
                    pstmt.executeUpdate();
                    JOptionPane.showMessageDialog(this, "Patient Added Successfully!");
                }
            } catch (SQLException ex) {
                JOptionPane.showMessageDialog(this, "Failed to add patient: " + ex.getMessage());
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Invalid input format!");
            }
        }
    }

    // Book Appointment Form
    private void showBookAppointmentForm(ActionEvent e) {
        JPanel panel = new JPanel(new GridLayout(5, 2, 10, 10));

        JComboBox<String> patientDropdown = new JComboBox<>();
        JComboBox<String> doctorDropdown = new JComboBox<>();
        JTextField dateField = new JTextField();
        JTextField timeField = new JTextField();

        // Load patients
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT patient_id, name FROM patients")) {
            while (rs.next()) {
                int id = rs.getInt("patient_id");
                String name = rs.getString("name");
                patientDropdown.addItem(id + " - " + name);
            }
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(this, "Failed to load patients: " + ex.getMessage());
        }

        // Load doctors
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT doctor_id, name FROM doctors")) {
            while (rs.next()) {
                int id = rs.getInt("doctor_id");
                String name = rs.getString("name");
                doctorDropdown.addItem(id + " - " + name);
            }
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(this, "Failed to load doctors: " + ex.getMessage());
        }

        panel.add(new JLabel("Patient:"));
        panel.add(patientDropdown);
        panel.add(new JLabel("Doctor:"));
        panel.add(doctorDropdown);
        panel.add(new JLabel("Date (YYYY-MM-DD):"));
        panel.add(dateField);
        panel.add(new JLabel("Time (HH:MM:SS):"));
        panel.add(timeField);

        int result = JOptionPane.showConfirmDialog(this, panel, "Book Appointment", JOptionPane.OK_CANCEL_OPTION);
        if (result == JOptionPane.OK_OPTION) {
            try {
                String selectedPatient = (String) patientDropdown.getSelectedItem();
                String selectedDoctor = (String) doctorDropdown.getSelectedItem();
                int patientId = Integer.parseInt(selectedPatient.split(" - ")[0]);
                int doctorId = Integer.parseInt(selectedDoctor.split(" - ")[0]);
                String date = dateField.getText();
                String time = timeField.getText();

                String sql = "INSERT INTO appointments (patient_id, doctor_id, appointment_date, appointment_time) VALUES (?, ?, ?, ?)";
                try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
                    pstmt.setInt(1, patientId);
                    pstmt.setInt(2, doctorId);
                    pstmt.setDate(3, Date.valueOf(date));
                    pstmt.setTime(4, Time.valueOf(time));
                    pstmt.executeUpdate();
                    JOptionPane.showMessageDialog(this, "Appointment Booked Successfully!");
                }
            } catch (SQLException ex) {
                JOptionPane.showMessageDialog(this, "Failed to book appointment: " + ex.getMessage());
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Invalid input format!");
            }
        }
    }

    // View Appointments Action (Table View)
    private void viewAppointmentsAction(ActionEvent e) {
        try {
            String sql = """
                SELECT a.appointment_id, p.name AS patient, d.name AS doctor, 
                       a.appointment_date, a.appointment_time
                FROM appointments a
                JOIN patients p ON a.patient_id = p.patient_id
                JOIN doctors d ON a.doctor_id = d.doctor_id
                """;
            try (Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery(sql)) {

                ResultSetMetaData metaData = rs.getMetaData();
                int columnCount = metaData.getColumnCount();
                String[] columnNames = new String[columnCount];
                for (int i = 1; i <= columnCount; i++) {
                    columnNames[i - 1] = metaData.getColumnName(i);
                }

                java.util.List<String[]> data = new java.util.ArrayList<>();
                while (rs.next()) {
                    String[] row = new String[columnCount];
                    for (int i = 1; i <= columnCount; i++) {
                        row[i - 1] = rs.getString(i);
                    }
                    data.add(row);
                }

                String[][] dataArray = data.toArray(new String[0][]);
                JTable table = new JTable(dataArray, columnNames);
                JScrollPane scrollPane = new JScrollPane(table);
                table.setFillsViewportHeight(true);

                JDialog dialog = new JDialog(this, "Appointments", true);
                dialog.setSize(600, 400);
                dialog.add(scrollPane);
                dialog.setLocationRelativeTo(this);
                dialog.setVisible(true);
            }
        } catch (SQLException ex) {
            JOptionPane.showMessageDialog(this, "Failed to fetch appointments: " + ex.getMessage());
        }
    }

    // Update Appointment Action
    private void updateAppointmentAction(ActionEvent e) {
        // Select Appointment to Update
        String appointmentIdStr = JOptionPane.showInputDialog(this, "Enter Appointment ID to Update:");
        if (appointmentIdStr != null && !appointmentIdStr.isEmpty()) {
            try {
                int appointmentId = Integer.parseInt(appointmentIdStr);
                String sql = "SELECT * FROM appointments WHERE appointment_id = ?";
                try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
                    pstmt.setInt(1, appointmentId);
                    ResultSet rs = pstmt.executeQuery();
                    if (rs.next()) {
                        String patientName = rs.getString("patient_id");
                        String doctorName = rs.getString("doctor_id");
                        String appointmentDate = rs.getString("appointment_date");
                        String appointmentTime = rs.getString("appointment_time");

                        // Update appointment form
                        JPanel panel = new JPanel(new GridLayout(4, 2, 10, 10));
                        JTextField doctorField = new JTextField(doctorName);
                        JTextField dateField = new JTextField(appointmentDate);
                        JTextField timeField = new JTextField(appointmentTime);

                        panel.add(new JLabel("Doctor:"));
                        panel.add(doctorField);
                        panel.add(new JLabel("Date:"));
                        panel.add(dateField);
                        panel.add(new JLabel("Time:"));
                        panel.add(timeField);

                        int result = JOptionPane.showConfirmDialog(this, panel, "Update Appointment", JOptionPane.OK_CANCEL_OPTION);
                        if (result == JOptionPane.OK_OPTION) {
                            String doctor = doctorField.getText();
                            String date = dateField.getText();
                            String time = timeField.getText();

                            String updateSql = "UPDATE appointments SET doctor_id = ?, appointment_date = ?, appointment_time = ? WHERE appointment_id = ?";
                            try (PreparedStatement updatePstmt = conn.prepareStatement(updateSql)) {
                                updatePstmt.setString(1, doctor);
                                updatePstmt.setString(2, date);
                                updatePstmt.setString(3, time);
                                updatePstmt.setInt(4, appointmentId);
                                updatePstmt.executeUpdate();
                                JOptionPane.showMessageDialog(this, "Appointment Updated Successfully!");
                            }
                        }
                    } else {
                        JOptionPane.showMessageDialog(this, "Appointment not found.");
                    }
                }
            } catch (SQLException | NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Failed to update appointment: " + ex.getMessage());
            }
        }
    }

    // Delete Appointment Action
    private void deleteAppointmentAction(ActionEvent e) {
        String appointmentIdStr = JOptionPane.showInputDialog(this, "Enter Appointment ID to Delete:");
        if (appointmentIdStr != null && !appointmentIdStr.isEmpty()) {
            try {
                int appointmentId = Integer.parseInt(appointmentIdStr);
                String sql = "DELETE FROM appointments WHERE appointment_id = ?";
                try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
                    pstmt.setInt(1, appointmentId);
                    int rowsAffected = pstmt.executeUpdate();
                    if (rowsAffected > 0) {
                        JOptionPane.showMessageDialog(this, "Appointment Deleted Successfully!");
                    } else {
                        JOptionPane.showMessageDialog(this, "Appointment not found.");
                    }
                }
            } catch (SQLException | NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Failed to delete appointment: " + ex.getMessage());
            }
        }
    }

    // Delete Patient Action
    private void deletePatientAction(ActionEvent e) {
        String patientIdStr = JOptionPane.showInputDialog(this, "Enter Patient ID to Delete:");
        if (patientIdStr != null && !patientIdStr.isEmpty()) {
            try {
                int patientId = Integer.parseInt(patientIdStr);

                // First, delete the appointments for this patient to avoid foreign key constraint issues
                String deleteAppointmentsSql = "DELETE FROM appointments WHERE patient_id = ?";
                try (PreparedStatement pstmt = conn.prepareStatement(deleteAppointmentsSql)) {
                    pstmt.setInt(1, patientId);
                    pstmt.executeUpdate();
                }

                // Then, delete the patient from the patient table
                String deletePatientSql = "DELETE FROM patients WHERE patient_id = ?";
                try (PreparedStatement pstmt = conn.prepareStatement(deletePatientSql)) {
                    pstmt.setInt(1, patientId);
                    int rowsAffected = pstmt.executeUpdate();
                    if (rowsAffected > 0) {
                        // Optionally, reset the auto-increment value to allow reuse of IDs (this is optional and depends on your database)
                        String resetAutoIncrementSql = "ALTER TABLE patients AUTO_INCREMENT = 1";
                        try (Statement stmt = conn.createStatement()) {
                            stmt.executeUpdate(resetAutoIncrementSql);
                        }
                        JOptionPane.showMessageDialog(this, "Patient Deleted Successfully!");
                    } else {
                        JOptionPane.showMessageDialog(this, "Patient not found.");
                    }
                }
            } catch (SQLException | NumberFormatException ex) {
                JOptionPane.showMessageDialog(this, "Failed to delete patient: " + ex.getMessage());
            }
        }
    }

    // Login Dialog (unchanged)
    private boolean login() {
        JPanel panel = new JPanel(new GridLayout(2, 2));
        JLabel userLabel = new JLabel("Username:");
        JLabel passLabel = new JLabel("Password:");
        JTextField userField = new JTextField();
        JPasswordField passField = new JPasswordField();

        panel.add(userLabel);
        panel.add(userField);
        panel.add(passLabel);
        panel.add(passField);

        int option = JOptionPane.showConfirmDialog(this, panel, "Login", JOptionPane.OK_CANCEL_OPTION, JOptionPane.PLAIN_MESSAGE);
        if (option == JOptionPane.OK_OPTION) {
            String username = userField.getText();
            String password = new String(passField.getPassword());
            String sql = "SELECT * FROM users WHERE username = ? AND password = ?";
            try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
                pstmt.setString(1, username);
                pstmt.setString(2, password);
                ResultSet rs = pstmt.executeQuery();
                return rs.next();
            } catch (SQLException ex) {
                JOptionPane.showMessageDialog(this, "Login failed: " + ex.getMessage());
            }
        }
        return false;
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            DentalClinicApp app = new DentalClinicApp();
            if (app.login()) {
                app.setVisible(true);
            } else {
                JOptionPane.showMessageDialog(null, "Login failed or canceled!", "Error", JOptionPane.ERROR_MESSAGE);
                System.exit(0);
            }
        });
    }
