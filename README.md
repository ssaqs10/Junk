# Junk
Store Snippets of Code to used for 

Java UDP sender (localhost)

import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.nio.charset.StandardCharsets;

public class aSender {

    public static void main(String[] args) {
        String host = "127.0.0.1";
        int port = 10110; 

        try (DatagramSocket socket = new DatagramSocket()) {

            String dptSentence = buildDptSentence(12.3, 0.0);

            byte[] data = (dptSentence + "\r\n")
                    .getBytes(StandardCharsets.US_ASCII);

            InetAddress address = InetAddress.getByName(host);

            DatagramPacket packet =
                    new DatagramPacket(data, data.length, address, port);

            socket.send(packet);

            System.out.println("Sent: " + dptSentence);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static String buildDptSentence(double depth, double offset) {
        String body = String.format("SDDPT,%.1f,%.1f", depth, offset);
        return "$" + body + "*" + checksum(body);
    }

    private static String checksum(String sentence) {
        int checksum = 0;
        for (char c : sentence.toCharArray()) {
            checksum ^= c;
        }
        return String.format("%02X", checksum);
    }
}


import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class NmeaDptUdpSender {

    private static final String HOST = "127.0.0.1";
    private static final int PORT = 10110;

    public static void main(String[] args) {
        DatagramSocket socket = null;
        ScheduledExecutorService scheduler = null;

        try {
            InetAddress address = InetAddress.getByName(HOST);
            socket = new DatagramSocket();

            scheduler = Executors.newSingleThreadScheduledExecutor();

            Runnable sendTask = () -> {
                try {
                    String message = buildNmeaDpt();
                    byte[] data = message.getBytes(StandardCharsets.US_ASCII);

                    DatagramPacket packet =
                            new DatagramPacket(data, data.length, address, PORT);

                    socket.send(packet);
                    System.out.println("Sent: " + message.trim());
                } catch (Exception e) {
                    e.printStackTrace();
                }
            };

            // Send every 500 milliseconds
            scheduler.scheduleAtFixedRate(
                    sendTask,
                    0,
                    500,
                    TimeUnit.MILLISECONDS
            );

        } catch (Exception e) {
            e.printStackTrace();
        }

        // Optional: shutdown hook for clean exit
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.out.println("Shutting down...");
            if (scheduler != null) scheduler.shutdown();
            if (socket != null) socket.close();
        }));
    }

    // Build NMEA DPT sentence
    private static String buildNmeaDpt() {
        double depth = 12.5;     // meters
        double offset = 0.0;     // transducer offset

        String body = String.format("SDDPT,%.1f,%.1f", depth, offset);
        String checksum = checksum(body);

        return "$" + body + "*" + checksum + "\r\n";
    }

    // Calculate NMEA checksum
    private static String checksum(String sentence) {
        int cs = 0;
        for (char c : sentence.toCharArray()) {
            cs ^= c;
        }
        return String.format("%02X", cs);
    }
}


