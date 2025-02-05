# sfstarter

document.addEventListener('keydown', function (event) {
    if (event.key === 'Enter') {
        const elements = document.getElementsByClassName('g-bubble');
        if (elements.length > 0) {
            elements[0].click(); // Triggers click event on the first matching element
        }
    }
});

