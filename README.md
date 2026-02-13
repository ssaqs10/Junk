

1. Message A → TCP Server 1


2. Message B → TCP Server 2


3. Another server receives both


4. It forwards them individually (as separate messages) to the same final server







Publisher
   ├──> Server1 (5001) --\
   ├──> Server2 (5002) ----> Relay Server (6000) ----> Final Server (7000)

The relay server simply forwards each message immediately.


---

✅ Publisher

import java.io.OutputStream;
import java.net.Socket;

public class DualPublisher {

    public static void main(String[] args) throws Exception {

        send("localhost", 5001, "Message A");
        send("localhost", 5002, "Message B");
    }

    private static void send(String host, int port, String message) throws Exception {
        try (Socket socket = new Socket(host, port);
             OutputStream os = socket.getOutputStream()) {

            os.write((message + "\n").getBytes());
            os.flush();
        }
    }
}


---

✅ Server1 & Server2 (Forward Immediately)



import java.io.*;
import java.net.*;

public class ForwardingServer {

    private static final String RELAY_HOST = "localhost";
    private static final int RELAY_PORT = 6000;

    public static void main(String[] args) throws Exception {

        int port = Integer.parseInt(args[0]);
        ServerSocket serverSocket = new ServerSocket(port);

        System.out.println("Listening on port " + port);

        while (true) {
            Socket client = serverSocket.accept();
            new Thread(() -> handle(client)).start();
        }
    }

    private static void handle(Socket client) {
        try (BufferedReader reader = new BufferedReader(
                new InputStreamReader(client.getInputStream()))) {

            String message = reader.readLine();
            System.out.println("Received: " + message);

            // Forward immediately to Relay
            try (Socket forward = new Socket(RELAY_HOST, RELAY_PORT);
                 OutputStream os = forward.getOutputStream()) {

                os.write((message + "\n").getBytes());
                os.flush();
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

Run:

java ForwardingServer 5001
java ForwardingServer 5002


---

✅ Relay Server (Just Pass Through – No Combine)

import java.io.*;
import java.net.*;

public class RelayServer {

    private static final int FINAL_SERVER_PORT = 7000;

    public static void main(String[] args) throws Exception {

        ServerSocket serverSocket = new ServerSocket(6000);
        System.out.println("Relay running on port 6000");

        while (true) {
            Socket socket = serverSocket.accept();
            new Thread(() -> handle(socket)).start();
        }
    }

    private static void handle(Socket socket) {
        try (BufferedReader reader = new BufferedReader(
                new InputStreamReader(socket.getInputStream()))) {

            String msg = reader.readLine();

            // Immediately forward WITHOUT combining
            try (Socket finalSocket = new Socket("localhost", FINAL_SERVER_PORT);
                 OutputStream os = finalSocket.getOutputStream()) {

                os.write((msg + "\n").getBytes());
                os.flush();
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}


---

✅ Final Server

import java.io.*;
import java.net.*;

public class FinalServer {

    public static void main(String[] args) throws Exception {

        ServerSocket serverSocket = new ServerSocket(7000);
        System.out.println("Final Server listening on port 7000");

        while (true) {
            Socket socket = serverSocket.accept();

            BufferedReader reader = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));

            String msg = reader.readLine();
            System.out.println("FINAL RECEIVED: " + msg);
        }
    }
}


---

🔥 Final Result

Output at Final Server:

FINAL RECEIVED: Message A
FINAL RECEIVED: Message B

Each message is:

Independent

Not merged

Not delayed

Immediately forwarded



---

If you want, I can now show you:

🔹 Version using one persistent connection (better performance)

🔹 Version that handles 1000+ messages per second

🔹 Version using ScheduledExecutor (since you worked with timed sending before)

🔹 Or version adapted for NMEA messages (DPT / MWV)
