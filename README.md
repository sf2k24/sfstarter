// Merge lines starting with "-"
  const merged: string[] = [];
  for (const line of disputeFlagsSample) {
    if (line.trim().startsWith("-")) {
      merged[merged.length - 1] += " " + line.trim();
    } else {
      merged.push(line);
    }
  }

  // Build lookup
  const map: Record<string, string> = {};
  for (const item of merged) {
    const [num, ...desc] = item.split(":");
    map[num.trim()] = desc.join(":").trim();
  }

  // Pick only selected flags
  this.flagStrings = {};
  disputeFlags?.forEach(f => {
    if (map[f]) this.flagStrings[f] = map[f];
  });
