# 🛰️ Network Router Simulator

This is a web-based, interactive network routing simulator. It provides a clean graphical user interface (GUI) for a console-based C program, allowing users to configure a static 4-router network, define attached network IPs, and simulate packet routing.

The simulator includes logic for **route history (caching)**, **direct-link detection**, and **hop-by-hop manual route definition**. It's built purely with HTML, CSS, and vanilla JavaScript, making it a lightweight and portable front-end project.

## ✨ Features

  * **Phase-Based UI:** A clean, two-phase process: (1) Configuration and (2) Routing.
  * **Static Topology:** Simulates a fixed 4-router topology with a hard-coded connection matrix.
  * **Dynamic Network Configuration:** Users can define the number of networks attached to each router and assign them unique, valid IP addresses.
  * **IP Validation:** All IP inputs are validated in real-time to ensure the correct IPv4 format (`x.x.x.x` where x is 0-255).
  * **Route History:** The simulator "learns" from successful routes. If a path between a source and destination IP is queried more than once, the cached path is instantly retrieved.
  * **Hop-by-Hop Manual Routing:** If no direct link exists, the user is guided through a manual, hop-by-hop path-building process, validating each step against the connection matrix.
  * **Direct Link Detection:** If a direct link exists between the source and destination routers, the user is given the choice to take the direct path or define a custom (longer) path.
  * **Real-time Logging:** All actions, validations, errors, and results are printed to a query log for clear, immediate feedback.

-----

## 🚀 How to Use

The application is simple to run. Just open the `index.html` file in any modern web browser.

### Phase 1: Configuration

Before you can start routing, you must configure the network.

1.  **View the Matrix:** Observe the **Router Connection Matrix**. This static matrix defines the "physical" links. For example, Router 1 (R1) can only talk directly to R1, R2, and R4.
2.  **Define Network Counts:** In the "Define Attached Networks" section, enter the number of networks (e.g., LANs) attached to each router (R1-R4).
3.  **Generate Fields:** Click the **"Generate IP Fields"** button. This will create the required number of text inputs.
4.  **Enter IP Addresses:** Enter a unique, valid IPv4 address for each network. The simulator uses these IPs to identify which router a "packet" is coming from or going to.
5.  **Save & Start:** Click **"Save Configuration & Start"**. This validates all IPs and, if successful, hides the configuration card and displays the routing simulation card.

### Phase 2: Routing Simulation

Once configured, you can begin querying the network.

1.  **Enter IPs:** Type a **Source IP** and a **Destination IP** into the input fields. These must be IPs you defined during the configuration phase.
2.  **Find Route:** Click the **"Find Route"** button.

#### Routing Scenarios

When you click "Find Route", one of three things will happen:

  * **Scenario A: History Found**

    > If you have routed between these two IPs before, the log will display a `[HISTORY FOUND]` message and instantly show the cached path (e.g., `124`).

  * **Scenario B: New Route (Direct Link Found)**

    > If this is a new query *and* the source router (e.g., R1) has a direct link to the destination router (e.g., R4), you will be given two buttons:

    >   * **Use Direct Path:** Automatically selects the `R1 -> R4` path, logs it, and saves it to history.
    >   * **Define Manual Path:** Hides the choice and opens the **Manual Route Definition** controls (see below). This allows you to define a longer path, like `R1 -> R2 -> R3 -> R4`.

  * **Scenario C: New Route (No Direct Link)**

    > If this is a new query *and* no direct link exists (e.g., R1 to R3), the **Manual Route Definition** controls will appear automatically.

#### Manual Route Definition

This UI guides you in building a valid path, hop by hop.

1.  **View State:** The controls show your **Current Path** (e.g., `1`), your **Current Router** (e.g., `R1`), and your target **Destination** (e.g., `R3`).
2.  **Enter Next Hop:** Enter the ID (1-4) of the *next* router you want to hop to.
3.  **Add Hop:** Click **"Add Hop"**.
      * **If Invalid:** An error will appear if you try to hop to a router that isn't connected (e.g., from R1 to R3) or to the current router (e.g., R1 to R1).
      * **If Valid:** The path is updated (e.g., `12`), and your "Current Router" becomes the new hop (e.g., `R2`).
4.  **Finalize Path:**
      * You continue adding hops until your *current* router has a direct link to the destination.
      * At this point (e.g., you are at R2, and the destination is R3), the UI will show an info message: `Info: R2 is now directly connected to Destination R3. Type 0 to finalize.`
      * Enter `0` and click **"Add Hop"** to complete the path.
5.  **Route Logged:** The final, valid path (e.g., `123`) is logged, displayed, and saved to the route history.

-----

## 💡 Project Logic & Implementation

This project is a direct JavaScript translation of a C console program's logic.

  * **State Management:** All application state (router IP configurations, route history) is stored in global JavaScript variables and a `Map` object. There is no backend or database.
  * **Connection Matrix:** The router topology is a static 2D array (matrix) hard-coded in `script.js`. This matrix is the "source of truth" for all path validation.
    ```javascript
    const connectionMatrix = [
        [1, 1, 0, 1], // R1
        [1, 1, 1, 0], // R2
        [0, 1, 1, 1], // R3
        [1, 0, 1, 1]  // R4
    ];
    ```
  * **Route History:** The route history (equivalent to C's `route_history` and `intermediate_history`) is implemented as a JavaScript `Map`.
      * **Key:** A string combining the source and destination IPs (`${sourceIp}*${destIp}`).
      * **Value:** A string representing the concatenated path (e.g., `"124"`).
        Using a Map provides efficient `O(1)` lookups, which simulates the C program's history-checking loop.
  * **Event-Driven Model:** Unlike the C program's `while` loop, the simulation is event-driven. The logic is paused and resumed using `click` event listeners, with the current routing state preserved in the `manualRouteState` object.

## 💻 Technology Stack

  * **HTML5:** Semantic HTML for structure and accessibility.
  * **CSS3:** Modern CSS with Flexbox/Grid for a clean, responsive layout.
  * **JavaScript (ES6+):** Vanilla JavaScript for all application logic, including DOM manipulation, validation, and state management.
