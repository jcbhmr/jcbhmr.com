---
title: Chunk Base map to CSV
---

I recently wanted to copy-paste all of the Chunk Base coordinates for every single POI on the map for a ±5000 block square around (0, 0). There is no (current) official way to download a Chunk Base map as a CSV file.

Here's how you can do it anyway:

1. Navigate to https://www.chunkbase.com/apps/seed-map
2. Enter your seed, version, etc.
3. Make sure the map shows everything you want to export
4. Open DevTools using `F12` or `Ctrl`+`Shift`+`I`
5. Type `allow pasting` if pasting is disabled
6. Copy & paste the script into your DevTools panel
7. Right-click & `Copy string contents` to get the CSV export and use it however you want to! 🎉

```js
const CSV = await import("https://esm.run/csv-string@4.1.1");
const table = [["Type", "X", "Y", "Z", "Details"]];
const oldOnPoiDrawn = CB3TooltipManager.onPoiDrawn;
CB3TooltipManager.onPoiDrawn = function (type, repr, coords, details) {
  const [x, y, z] = coords;
  table.push([type, x, y, z, JSON.stringify(details)]);
};
CB3FinderApp.trigger("redrawmap");
CB3TooltipManager.onPoiDrawn = oldOnPoiDrawn;
CSV.stringify(table);
```

![screenshot of me doing that](/uploads/2024-12-05-001.png)

## How it works

The real trick is intercepting the `onPoiDrawn` event and using the provided structured data to construct our own CSV table instead of drawing images to the map canvas. We really lucked out that `CB3FinderApp` and friends are all global variables!

So we:

1. Create our table. It has `Type`, `X`, `Y`, `Z`, and `Details` columns.
2. Stash the old `onPoiDrawn` function that probably does some important things.
3. **Overwrite** the `onPoiDrawn` function with our own that `.push()`-es a new row for each POI that needs to be drawn. You can `console.log(arguments)` to see the complete arguments list. This was the most complicated part; you need to inspect the `arguments` in the console and then pluck the right attributes off the right objects to get the data you want.
4. Trigger a redraw. This was another case of trial and error. You can `console.log(CB3FinderApp)` in the console and look for any event-related properties to find more events. `redrawmap` seemed like a good thing to try and it worked! There may be other events that do different things. Who knows!
5. Reset the `onPoiDrawn` callback back to the original.
6. Return the table stringified as CSV. Since this is the last evaluated expression it will be shown to the user in the DevTools console. Chrome (no idea about other browsers) has a handy `Copy string contents` option in the right-click menu which can be used to easily copy the CSV string to your clipboard.

You can very easily edit the code and export the POI data in JSON or some other format if you like.
