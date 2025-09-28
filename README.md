```javascript

const map = {};
let currentFlag = "";

for (const line of disputeFlagsSample) {
  if (/^\d+:/.test(line)) {
    // Update current flag number
    currentFlag = line.split(":")[0].trim();
  } else if (line.trim().startsWith("-") && currentFlag) {
    // Map only "-" lines
    map[currentFlag] = line.trim();
  }
}

// Pick only requested flags
this.flagStrings = {};
disputeFlags?.forEach(f => {
  if (map[f]) this.flagStrings[f] = map[f];
});

console.log("this.flagStrings", this.flagStrings);
```
