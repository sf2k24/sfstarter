# sfstarter


function isValidDate(dateString, format = "YYYY-MM-DD") {
    let regex;
    if (format === "YYYY-MM-DD") {
        regex = /^\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])$/;
    } else if (format === "MM/DD/YYYY") {
        regex = /^(0[1-9]|1[0-2])\/(0[1-9]|[12]\d|3[01])\/\d{4}$/;
    } else if (format === "DD-MM-YYYY") {
        regex = /^(0[1-9]|[12]\d|3[01])-(0[1-9]|1[0-2])-\d{4}$/;
    } else {
        return false; // Unsupported format
    }

    if (!regex.test(dateString)) {
        return false;
    }

    // Additional logical validation
    const [year, month, day] = dateString.split(/[-\/]/).map(Number);
    const date = new Date(year, month - 1, day);
    return date.getFullYear() === year && date.getMonth() === month - 1 && date.getDate() === day;
}

console.log(isValidDate("2023-02-28", "YYYY-MM-DD")); // true
console.log(isValidDate("2023-02-30", "YYYY-MM-DD")); // false
console.log(isValidDate("12/31/2023", "MM/DD/YYYY")); // true
console.log(isValidDate("31-12-2023", "DD-MM-YYYY")); // true
