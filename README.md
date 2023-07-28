# Profanity filter

File [filters.json](./filters.json) has different filter types.
The moderation script fetches this file every 5 minutes.
This README.md file is kept up-to-date with any changes to the moderation script.

### Index

- [Set up Visual Studio Code](#setup-vs-code)<br/>
- [Filters](#filters)<br/>
  - [Message filter](#message-filter)<br/>
  - [Name filter](#name-filter)<br/>
  - [Hard name filter](#hard-name-filter)<br/>
- [Whitelist](#whitelist)<br/>
- [Collection](#collection)<br/>
- [String builder](#string-builder)<br/>
- [Variations logic](#variations-logic)

## Setup VS Code

Download and install [Visual Studio Code](https://code.visualstudio.com/download) and [Git](https://git-scm.com/downloads). Then follow [these instructions](https://code.visualstudio.com/docs/sourcecontrol/intro-to-git#_set-up-git-in-vs-code) on how to set up this repository locally.

Once you cloned this repository locally, open it in its workspace. This is the case if project title in the file explorer (left menu) is suffixed with `(WORKSPACE)`. If this is not the case, close the project and reopen it by double clicking the file `profanity.code-workspace`.

## Filters

The JSON file contains 3 filters: `message`; `name`; and `hardName`. Each filter contains arrays with strings and string builders that are categorised by the properties: `notify`; `kick`; and `ban`. If you add (variations of) profanity that might result in a false positive, then you can add certain words to the `whitelist`.

### Message filter

The `message` filter is applied to both player messages and names.
This filter contains 3 properties: `notify`; `kick`; and `ban`.
Each property is an array that has both strings and string builder arrays.

### Name filter

The `name` filter is applied to player name only.
This filter contains 3 properties: `notify`; `kick`; and `ban`.
Each property is an array that has both strings and string builder arrays.

Example of detecting part of a player name: 'I Am Gay' → 'Gay' → 'gay'

### Hard name filter

The `hardName` filter is applied to player names only and
reads through the name by use of a [Regular Expression](https://regexr.com/) (RegExp).
All strings as used as-is; you cannot use RegExp escape characters, operators or other logic.
This filter contains 3 properties: `notify`; `kick`; and `ban`.
Each property is an array that has both strings and string builder arrays.
Using a string builder is more (CPU) efficient as opposed to manually typing out string variations.

Example of detecting part of a player name: 'Me A Gaymer' → 'MeAGaymer' → 'Gay' → 'gay'.

## Whitelist

The `whitelist` filter prevents false positives when finding profanity.
If the `message`, `name` or `hardName` filter finds profanity in a string
then the `whitelist` filter will check whether the player input is whitelisted.
The profanity is ignored if (part of) the player input is whitelisted.
Read paragraph [profanity variations](#profanity-variations) for more information on how profanity is found.

```Typescript
const whitelist: string[] = [ "sheet", "45s", "plss", "penne" ];
const message: string = "The baby boom started in the 45s";

/**
 * Read paragraph 'Profanity variations' to learn
 * more about how profanity is found in strings.
 */
const foundProfanity: string = "45s"; // "45s" → "ass"
if (whitelist.includes(foundProfanity) {
  // "45s" is whitelisted so profanity "45s" is ignored.
}
```

## Collection

The `collection` property is an object with key-value pairs used in the string building process.
Register a key with the value of a string array to use as a reference in the nested string arrays of string builders.
Each key is case sensitive and must be unique enough not to be accidentally referenced.

```Typescript
// Example collection key-pairs.
const collection: Record<string, string[]> = {
  colourList: ["black", "brown", "yellow", "white"],
  humansList: ["men", "women"],
};

// Example profanity in player messages.
const messageProfanity: (string | (string | string[])[])[] = [
  "fuckyou",
  "killyourself",
  [
    ["kill", "murder"],
    "colourList",
    ["humansList", "humans"]
  ],
  "fuckme"
];

// Variable `messageProfanity` will be solved. Result:
const messageProfanity: (string | (string | string[])[])[] = [
  "fuckyou",
  "killyourself",
  [
    ["kill", "murder"],
    ["black", "brown", "yellow", "white"], // 4 strings inserted
    ["men", "women", "humans"] // 2 strings inserted
  ],
  "fuckme"
]
```
**Warning:** common worded keys may result in accidental referencing. Example: use `humansList` instead of `humans`; or `raceExtended` instead of `race`.

## String builder

The `stringBuilders` property is an array with _string builder_ objects.
This property is an alternative to nesting your string builder arrays in the profanity arrays.
One advantage is that you can set your own separator, which is an empty string by default.

```Typescript
interface StringBuilder {
  version: "basic" | "extended",
  filter: "message" | "name" | "hardName" | "whitelist",
  modAction: "notify" | "kick" | "ban",
  separator: string,
  builder: (string | string[])[],
};

const stringBuilders: StringBuilder[] = [
  {
    version: "basic",
    filter: "name",
    modAction: "kick",
    separator: " ",
    builder: [
      ["adolf", "adilf", "adulf", "adelf", "adalf"],
      ["hit", "hat", "hut", "het", "hat"],
      ["ler", "lar"]
    ]
  }
];

/**
 * A script creates and adds the built string builder
 * to the correct versioned filter type.
 */
const profanityList: string[] = [
  "adolf hitler", "adilf hitler", "adulf hitler", "adelf hitler", "adalf hitler",
  "adolf hatler", "adilf hatler", "adulf hatler", "adelf hatler", "adalf hatler",
  "adolf hutler", "adilf hutler", "adulf hutler", "adelf hutler", "adalf hutler",
  "adolf hetler", "adilf hetler", "adulf hetler", "adelf hetler", "adalf hetler",
  "adolf hatler", "adilf hatler", "adulf hatler", "adelf hatler", "adalf hatler",
  "adolf hitlar", "adilf hitlar", "adulf hitlar", "adelf hitlar", "adalf hitlar",
  "adolf hatlar", "adilf hatlar", "adulf hatlar", "adelf hatlar", "adalf hatlar",
  "adolf hutlar", "adilf hutlar", "adulf hutlar", "adelf hutlar", "adalf hutlar",
  "adolf hetlar", "adilf hetlar", "adulf hetlar", "adelf hetlar", "adalf hetlar",
  "adolf hatlar", "adilf hatlar", "adulf hatlar", "adelf hatlar", "adalf hatlar",
]
```

## Variations logic

The script creates variations of all player input strings.
This block of code is a simplified version of such variations to an idea how profanity is detected.

```Typescript
enum ModAction {
  notify,
  kick,
  ban,
};

interface MessageFilter extends Record<keyof typeof ModAction, string[]> { };

// Example message filter.
const messageFilter: MessageFilter = {
  notify: ["zoom meeting", "thicc girl"],
  kick: ["penis", "www.bit", "bitch"],
  ban: ["hugepenis", "bigpenis"],
};

// Example player message input.
const playerMessage: string = "Suck my big pen1s, Jack!";
const slices: string[] = [
  "Suck", "my", "big", "pen1s,", "Jack!",
  "Suck my", "my big", "big pen1s,", "pen1s, Jack!",
  "Suck my big", "my big pen1s,", "my pen1s, Jack!",
  "Suck my big pen1s,", "my big pen1s, Jack!",
  "Suck my big pen1s, Jack!"
];

// Loop over the `slices`. For this example, we take `slices[7]`.
const slice: string = "big pen1s,";

const variations: string[] = (function createVariations() {
  const array: string[] = [
    slice,
    Purifier.numberToLetter(slice) // replace numbers with letters. Example: 1 → i
  ];

  for (const s of array) array.push(
    s.replace(/\W/g, ""),
    s.replace(/\d/g, "")
  );

  for (const s of array) array.push(
    s.replace(/\s/g, "");
  );

  // Remove duplicates (removes 4 elements in this example)
  return array.filter((v, i, arr) => arr.indexOf(v) === i);
})();

// Check if any variation matches any profanity.
for (const variation of variations) {
  for (const key of Object.keys(messageFilter)) {
    if (messageFilter[key].includes(variation)) {
      console.log("Profanity found: " + variation);
    }
  }
}

console.log(variations);
// Output: [ "big pen1s,", "big penis,", "big penis", "big pens",
//           "bigpen1s,",  "bigpenis,",  "bigpenis",  "bigpens" ]
```