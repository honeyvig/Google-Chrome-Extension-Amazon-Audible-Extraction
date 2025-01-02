# Google-Chrome-Extension-Amazon-Audible-Extraction
- Google chrome is required
- Download Amazon Audible BSR extension
- I will give you categories to go through
- Find out the BSR
- Add the data of books that are ranked 8,000-30,000 ONLY
- Then filter through those and only keep the books of individuals that are NOT famous or REAL AUTHORS 
------------------
Creating a Google Chrome extension to interact with Amazon Audible and filter books based on their Best Seller Rank (BSR) while excluding famous authors requires several key steps. Below is the structured approach for building this extension:
Overview:

This Chrome extension will help users extract data on books from Amazon Audible. The goal is to retrieve the BSR (Best Seller Rank) of books, and filter them based on two main criteria:

    BSR: Only books with a BSR between 8,000-30,000 will be selected.
    Author Type: Only books by non-famous or independent authors will be retained, excluding well-known authors.

Features:

    Fetch Data: Fetch book data from Amazon Audible.
    BSR Filtering: Only books ranked between 8,000 and 30,000 in the BSR will be saved.
    Author Filtering: Exclude books written by famous authors (this can be done by creating a list of famous authors or using other heuristics).
    Download Option: Save the filtered list of books (in CSV, JSON, etc.).
    Category-Specific: Allow filtering by categories (you will provide the categories).

Technical Steps:

    Manifest File (manifest.json):
        Set up the Chrome extension with necessary permissions and background scripts.

{
  "manifest_version": 3,
  "name": "Amazon Audible BSR Fetcher",
  "description": "Fetch and filter Amazon Audible books based on BSR and author type.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://www.audible.com/*"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "images/icon16.png",
      "48": "images/icon48.png",
      "128": "images/icon128.png"
    }
  },
  "host_permissions": [
    "https://www.audible.com/*"
  ]
}

    Content Script (content.js):
        Extract data from the Amazon Audible website.
        Identify the BSR and filter books based on the specified range (8,000-30,000).
        Filter books based on authors (this could be a manual list of famous authors or heuristics like checking if the author name appears on a famous author's list).

// content.js
const famousAuthors = ['Stephen King', 'J.K. Rowling', 'Dan Brown']; // Example list of famous authors

// Function to extract BSR and author information
function extractBookData() {
  const books = [];
  
  // Loop through book elements on the page (e.g., using a class or selector)
  const bookElements = document.querySelectorAll('.product-list .product'); // Adjust selector based on actual structure

  bookElements.forEach((book) => {
    const bsrElement = book.querySelector('.bsr-rank'); // Adjust selector based on actual structure
    const authorElement = book.querySelector('.author-name'); // Adjust selector based on actual structure
    
    if (bsrElement && authorElement) {
      const bsr = parseInt(bsrElement.textContent.replace(/[^\d]/g, ''), 10);
      const author = authorElement.textContent.trim();
      
      if (bsr >= 8000 && bsr <= 30000 && !famousAuthors.includes(author)) {
        const title = book.querySelector('.product-title').textContent.trim(); // Adjust selector as needed
        books.push({ title, author, bsr });
      }
    }
  });

  return books;
}

// Send the extracted data to the background or popup
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === "fetchBooks") {
    const bookData = extractBookData();
    sendResponse({ books: bookData });
  }
});

    Background Script (background.js):
        Handles communication with the content script and stores the data.

// background.js
chrome.runtime.onInstalled.addListener(() => {
  console.log("Amazon Audible BSR Fetcher extension installed.");
});

// Listen for messages from the content script or popup
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === "fetchBooks") {
    chrome.scripting.executeScript({
      target: { tabId: sender.tab.id },
      func: extractBookData
    }, (result) => {
      sendResponse({ books: result[0].result });
    });
  }
  return true; // Keep the message channel open for sendResponse
});

    Popup HTML (popup.html):
        A simple UI to allow the user to trigger the data extraction process and view/download results.

<!DOCTYPE html>
<html>
  <head>
    <title>Amazon Audible BSR Fetcher</title>
    <style>
      body {
        width: 300px;
        font-family: Arial, sans-serif;
      }
      button {
        margin-top: 20px;
        padding: 10px;
        background-color: #4CAF50;
        color: white;
        border: none;
        cursor: pointer;
      }
      #result {
        margin-top: 20px;
        font-size: 12px;
        color: #555;
      }
    </style>
  </head>
  <body>
    <h3>Fetch Audible BSR Data</h3>
    <button id="fetchData">Fetch Books</button>
    <div id="result"></div>

    <script src="popup.js"></script>
  </body>
</html>

    Popup JavaScript (popup.js):
        This script handles user interaction and fetches data when the user clicks the "Fetch Books" button.

// popup.js
document.getElementById('fetchData').addEventListener('click', () => {
  chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
    chrome.scripting.executeScript({
      target: { tabId: tabs[0].id },
      func: extractBookData
    }, (result) => {
      const books = result[0].result;
      const resultDiv = document.getElementById('result');
      
      if (books.length > 0) {
        resultDiv.innerHTML = `<p>Found ${books.length} books with BSR between 8000 and 30000.</p>`;
        // Optionally display books or allow download
      } else {
        resultDiv.innerHTML = "<p>No books found in this range.</p>";
      }
    });
  });
});

Optional Enhancements:

    Export Data: Allow the user to export the filtered books list as a CSV or JSON file.
    Categorization: Implement a dropdown to select a category (if the page contains categories).
    Pagination: Handle pagination for pages with multiple book listings.
    Dynamic Author Check: Use an API or external database to dynamically check if the author is famous.

How the Extension Works:

    User clicks on the extension to open the popup.
    Popup triggers the content script to fetch book data from the current Audible page.
    Books are filtered based on BSR and author criteria (only books with a BSR between 8,000 and 30,000, and non-famous authors).
    Results are displayed to the user in the popup or exported to a file.

Conclusion:

This Chrome extension provides a streamlined way to gather and filter book data from Amazon Audible based on BSR and author type. You can customize the list of famous authors or use more sophisticated methods to automatically detect them. The extension is lightweight and can be easily extended with additional features like data export and pagination handling.
