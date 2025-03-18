const XLSX = require('xlsx');
const fs = require('fs');

// Sample JSON object
const jsonData = [
    { name: "Alice", age: 25, city: "New York" },
    { name: "Bob", age: 30, city: "Los Angeles" },
    { name: "Charlie", age: 35, city: "Chicago" }
];

// Convert JSON to worksheet
const worksheet = XLSX.utils.json_to_sheet(jsonData);

// Create a new workbook and append the worksheet
const workbook = XLSX.utils.book_new();
XLSX.utils.book_append_sheet(workbook, worksheet, "Sheet1");

// Write to an Excel file
XLSX.writeFile(workbook, "output.xlsx");

console.log("Excel file created successfully!");
