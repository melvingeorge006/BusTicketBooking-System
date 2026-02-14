# Bus-ticket-booking-system

Team Members
Joseph Toby - 24UBC239
Melvin George - 24UBC248

Description:

The Bus Ticket Booking System is a Java Swing–based application that allows users to book bus tickets through a graphical interface. It supports AC and Non-AC buses with separate seat management, real-time seat availability, ticket price calculation, and displays all booking details while demonstrating core object-oriented programming concepts.


program:
import javax.swing.*;
import java.awt.*;
import java.sql.*;
import java.util.*;

abstract class Ticket {
    protected String name;
    protected String seat;
    protected String busType;
    protected int price;

    Ticket(String name, String seat, String busType, int price) {
        this.name = name;
        this.seat = seat;
        this.busType = busType;
        this.price = price;
    }

    public String getTicketBlock() {
        return  "Passenger : " + name + "        Seat : " + seat + "\n" +
                "Type      : " + busType + "        Price: ₹" + price + "\n" +
                "---------------------------------------------\n";
    }
}

class ACTicket extends Ticket {
    ACTicket(String name, String seat) {
        super(name, seat, "AC", 500);
    }
}

class NonACTicket extends Ticket {
    NonACTicket(String name, String seat) {
        super(name, seat, "Non-AC", 300);
    }
}

public class BusTicketBookingSystem extends JFrame {

    private JTextField nameField;
    private JComboBox<String> busTypeBox;
    private JTextArea ticketArea;
    private JPanel seatPanel;

    private String selectedSeat = "";

    private Set<String> bookedACSeats = new HashSet<>();
    private Set<String> bookedNonACSeats = new HashSet<>();
    private java.util.List<Ticket> allTickets = new ArrayList<>();

    private Connection connection;

    public BusTicketBookingSystem() {

        initializeDatabase();   // 🔥 DB auto-init
        loadBookingsFromDB();   // 🔥 Load existing data

        setTitle("Bus Ticket Booking System");
        setSize(1200, 650);
        setLocationRelativeTo(null);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLayout(new BorderLayout(10,10));

        JLabel header = new JLabel("Bus Ticket Booking System", JLabel.CENTER);
        header.setFont(new Font("Segoe UI", Font.BOLD, 26));
        add(header, BorderLayout.NORTH);

        JPanel formPanel = new JPanel(new GridBagLayout());
        formPanel.setBorder(BorderFactory.createTitledBorder("Booking Details"));

        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(6,6,6,6);
        gbc.fill = GridBagConstraints.HORIZONTAL;

        nameField = new JTextField(14);
        busTypeBox = new JComboBox<>(new String[]{"AC", "Non-AC"});
        JButton bookBtn = new JButton("Confirm Booking");

        gbc.gridx = 0; gbc.gridy = 0;
        formPanel.add(new JLabel("Passenger Name:"), gbc);
        gbc.gridx = 1;
        formPanel.add(nameField, gbc);

        gbc.gridx = 0; gbc.gridy = 1;
        formPanel.add(new JLabel("Bus Type:"), gbc);
        gbc.gridx = 1;
        formPanel.add(busTypeBox, gbc);

        gbc.gridx = 1; gbc.gridy = 2;
        formPanel.add(bookBtn, gbc);

        add(formPanel, BorderLayout.WEST);

        seatPanel = new JPanel();
        seatPanel.setBorder(BorderFactory.createTitledBorder("Seat Layout"));
        add(seatPanel, BorderLayout.CENTER);

        ticketArea = new JTextArea();
        ticketArea.setEditable(false);
        ticketArea.setFont(new Font("Monospaced", Font.BOLD, 13));
        ticketArea.setBorder(BorderFactory.createTitledBorder("All Booked Tickets"));
        ticketArea.setPreferredSize(new Dimension(450, 0));

        add(ticketArea, BorderLayout.EAST);

        busTypeBox.addActionListener(e -> loadSeats());
        bookBtn.addActionListener(e -> bookTicket());

        loadSeats();
        updateTicketArea();
        setVisible(true);
    }

    /* ================= DATABASE INITIALIZATION ================= */

    private void initializeDatabase() {
        try {
            // Connect without database first
            Connection tempConn = DriverManager.getConnection(
                    "jdbc:mysql://localhost:3306/?serverTimezone=UTC",
                    "root",
                    "password"  // 🔁 Change this
            );

            Statement stmt = tempConn.createStatement();
            stmt.executeUpdate("CREATE DATABASE IF NOT EXISTS bus_booking_db");
            tempConn.close();

            connection = DriverManager.getConnection(
                    "jdbc:mysql://localhost:3306/bus_booking_db?serverTimezone=UTC",
                    "root",
                    "password"  // 🔁 Change this
            );

            Statement tableStmt = connection.createStatement();
            tableStmt.executeUpdate(
                    "CREATE TABLE IF NOT EXISTS tickets (" +
                    "id INT AUTO_INCREMENT PRIMARY KEY," +
                    "name VARCHAR(100)," +
                    "seat VARCHAR(10)," +
                    "busType VARCHAR(20)," +
                    "price INT)"
            );

        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, "Database Error: " + e.getMessage());
        }
    }

    /* ================= LOAD FROM DATABASE ================= */

    private void loadBookingsFromDB() {
        try {
            if (connection == null) return;

            Statement stmt = connection.createStatement();
            ResultSet rs = stmt.executeQuery("SELECT * FROM tickets");

            while (rs.next()) {
                String name = rs.getString("name");
                String seat = rs.getString("seat");
                String type = rs.getString("busType");

                Ticket ticket = type.equals("AC")
                        ? new ACTicket(name, seat)
                        : new NonACTicket(name, seat);

                allTickets.add(ticket);

                if (type.equals("AC"))
                    bookedACSeats.add(seat);
                else
                    bookedNonACSeats.add(seat);
            }

        } catch (Exception e) {
            System.out.println("Load error: " + e.getMessage());
        }
    }

    /* ================= SAVE TO DATABASE ================= */

    private void saveTicketToDB(Ticket ticket) {
        try {
            PreparedStatement ps = connection.prepareStatement(
                    "INSERT INTO tickets(name, seat, busType, price) VALUES (?, ?, ?, ?)"
            );

            ps.setString(1, ticket.name);
            ps.setString(2, ticket.seat);
            ps.setString(3, ticket.busType);
            ps.setInt(4, ticket.price);

            ps.executeUpdate();

        } catch (Exception e) {
            System.out.println("Save error: " + e.getMessage());
        }
    }

    /* ================= LOAD SEATS ================= */

    private void loadSeats() {
        seatPanel.removeAll();
        seatPanel.setLayout(new GridLayout(8, 5, 6, 6));

        boolean isAC = busTypeBox.getSelectedItem().equals("AC");
        Set<String> bookedSeats = isAC ? bookedACSeats : bookedNonACSeats;

        for (int i = 1; i <= 40; i++) {
            String seat = "S" + i;
            JButton btn = new JButton(seat);

            if (bookedSeats.contains(seat)) {
                btn.setBackground(Color.RED);
                btn.setEnabled(false);
            } else {
                btn.setBackground(Color.GREEN);
            }

            btn.addActionListener(e -> selectedSeat = seat);
            seatPanel.add(btn);
        }

        seatPanel.revalidate();
        seatPanel.repaint();
    }

    /* ================= BOOK TICKET ================= */

    private void bookTicket() {
        try {
            String name = nameField.getText().trim();
            if (name.isEmpty())
                throw new Exception("Enter passenger name");

            if (selectedSeat.isEmpty())
                throw new Exception("Select a seat");

            String type = busTypeBox.getSelectedItem().toString();
            Ticket ticket;

            if (type.equals("AC")) {
                bookedACSeats.add(selectedSeat);
                ticket = new ACTicket(name, selectedSeat);
            } else {
                bookedNonACSeats.add(selectedSeat);
                ticket = new NonACTicket(name, selectedSeat);
            }

            allTickets.add(ticket);
            saveTicketToDB(ticket);  // 🔥 Save to DB
            updateTicketArea();

            nameField.setText("");
            selectedSeat = "";
            loadSeats();

        } catch (Exception ex) {
            JOptionPane.showMessageDialog(this, ex.getMessage());
        }
    }

    /* ================= UPDATE TICKET AREA ================= */

    private void updateTicketArea() {
        StringBuilder sb = new StringBuilder();
        for (Ticket t : allTickets) {
            sb.append(t.getTicketBlock());
        }
        ticketArea.setText(sb.toString());
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(BusTicketBookingSystem::new);
    }
}
