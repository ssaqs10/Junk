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


