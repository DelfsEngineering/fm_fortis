# Fortis Elements Tokenization for FileMaker

This project provides an HTML page (`index.html`) designed to be used within a FileMaker Web Viewer to securely capture payment card details and obtain a token (`ticket_id`) using Fortis Elements. It uses Alpine.js for simple interactivity and the Fortis Commerce.js library to render the secure iframe.

## Features

*   **PCI Compliance:** Leverages Fortis Elements to ensure sensitive card data never touches your application directly.
*   **Tokenization:** Captures card details and exchanges them for a secure `ticket_id` (token) from Fortis.
*   **FileMaker Integration:** Designed specifically to receive necessary data from FileMaker and return the resulting `ticket_id` back to a FileMaker script.
*   **Client-Side Focus:** Handles the rendering of the payment form and the callback logic within the client-side web viewer environment.

## Technology Stack

*   HTML
*   Alpine.js (v3)
*   Fortis Commerce.js (v1.0.0)
*   FileMaker (as the host application)

## Workflow

1.  **Initiate (FileMaker):** A FileMaker script or action triggers the payment process.
2.  **Get Client Token (Server-Side):** Your FileMaker solution (or an intermediary service called by FileMaker) must make a **secure, server-side** `POST` request to the Fortis API endpoint `/v1/elements/ticket/intention`. This requires your Fortis API credentials and appropriate parameters (like `location_id`, `contact_id`). Fortis responds with a `client_token`.
    *   **Security Note:** This step **must** be handled server-side (e.g., using FileMaker's `Insert from URL` with cURL options, a microservice, or a backend script) to avoid exposing your API credentials in the client-side code.
3.  **Load Web Viewer (FileMaker):** FileMaker loads the `index.html` page into a Web Viewer object. It **must** pass the `client_token` obtained in Step 2 as a URL parameter named `token`.
    *   Example URL: `https://yourserver.com/path/to/index.html?token=FTcs_xxxxxxxxxxxxxxxxxxxx`
    *   Alternatively, if hosting isn't feasible, the HTML can be embedded directly in the Web Viewer calculation using a `data:` URL, substituting the token value within the HTML string.
4.  **Render Form (`index.html`):** The `index.html` page uses Alpine.js (`init()` function):
    *   Extracts the `client_token` from the URL.
    *   Initializes Fortis Commerce.js (`Commerce.elements`) with the token.
    *   Renders the secure Fortis Elements payment iframe inside the `<div id="payment">`.
5.  **User Input:** The user enters their card details directly into the secure Fortis iframe.
6.  **Tokenization (`index.html`):** When the user submits the form within the iframe:
    *   Fortis processes the details.
    *   The `Commerce.elements` instance emits a `done` event.
    *   The JavaScript listens for this `done` event and extracts the `ticket_id` from the event payload (`payload.data.id`).
7.  **Callback to FileMaker (`index.html`):**
    *   The JavaScript constructs an `fmp://` URL. This URL targets the currently open FileMaker file (`$`) and specifies a script to run, passing the `ticket_id` as a parameter.
    *   Example `fmp://` URL: `fmp://$/?script=HandleFortisTicket&param=FTts_yyyyyyyyyyyyyyyyyyyy`
    *   `window.location.href` is set to this URL, triggering FileMaker to run the specified script.
8.  **Process Token (FileMaker):** The designated FileMaker script (`HandleFortisTicket` in the example) receives the `ticket_id` as its script parameter. You can now store this token securely and use it for future transactions via the Fortis API according to their documentation.

## Setup

1.  **Server-Side Component:** Ensure you have a reliable method within your FileMaker solution (or an associated backend) to securely call the Fortis `/v1/elements/ticket/intention` endpoint and retrieve the `client_token`.
2.  **`index.html`:**
    *   Place the `index.html` file on a web server accessible by your FileMaker clients, or prepare it for embedding in the Web Viewer calculation.
    *   **Customize Callback Script Name:** Modify the `fmp://` URL generation within the `<script>` block in `index.html` to use the correct FileMaker script name you want to trigger (e.g., replace `HandleFortisTicket` with your script's name).
3.  **FileMaker:**
    *   Create the FileMaker script that handles the server-side call (Step 2).
    *   Configure the Web Viewer object to load `index.html` with the `client_token` (Step 3).
    *   Create the FileMaker script that will be called back by the `fmp://` URL (e.g., `HandleFortisTicket`) to receive and process the `ticket_id`.

## Files

*   `index.html`: The main HTML page containing the Alpine.js logic and Fortis Elements integration.
*   `README.md`: This documentation file. 
