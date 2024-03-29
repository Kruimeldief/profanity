# Profanity filter

File [filters.json](./filters.json) has different filter types.
The moderation script fetches this file every 5 minutes.
This README.md file is kept up-to-date with any changes to the moderation script.

### Index

• [Word filter](#word-filter)<br/>
• [Name filter](#name-filter)<br/>
• [Hard name filter](#hard-name-filter)<br/>
• [Whitelist](#whitelist)<br/>
• [Sentence filter](#sentence-filter)<br/>
• [Profanity variations](#profanity-variations)<br/>
• [Mod actions](#mod-actions)


### Word filter

The wordFilter is applied to both player messages and names.

### Name filter

The nameFilter is applied to player names only to allow players more freedom to talk about certain things.<br/>
Example of a player name: 'I Am Gay' → 'Gay' → 'gay'

### Hard name filter

The hardNameFilter is applied to player names only.
It reads through the name by use of a Regular Expression.<br/>
Example of a player name: 'Me A GayMer' → 'MeAGayMer' → 'Gay' → 'gay'.

### Whitelist

Whitelisted words prevent false positives when finding profanity.
If a profane part of a string contains a whitelisted word, that profane part is ignored.
Read paragraph [profanity variations](#profanity-variations) for more information about how profanity and whitelisted words are found.
```Javascript
const whitelist = [ "sheet", "45s", "plss", "penne" ];
const message = "The baby boom started in the 45s";

/**
 * Read paragraph 'Profanity variations' to learn
 * more about how profanity is found in strings.
 */
const foundProfanity = "45s"; // "45s" → "ass"
if (whitelist.includes(foundProfanity) {
  // "45s" is whitelisted so profanity "45s" is ignored.
}
```

### Sentence filter

The sentenceFilter has a list of objects that facilitate creating variations.<br/>
• `filter` is where the variations are added: wordFilter or nameFilter.<br/>
• `modAction` specifies what happens if a message contains the profane variation: KICK or SOFT_BAN (case sensitive).<br/>
• `list` is a nested array from which one word is selected per array to create the variation.<br/>
```Javascript
/**
 * If you leave an string empty in the list's nested array,
 * then you make the words in that array optional.
 * 
 * Each listed word is separated by a space.
 */
const example = {
  "filter" : "nameFilter",
  "modAction" : "KICK",
  "list" : [
    [ "suck", "sug", "lick", "lig" ],
    [ "", "my", "ma" ],
    [ "cock", "balls", "nuts" ]
  ]
};

/**
 * The moderation script creates and adds the
 * following profanity strings to the nameFilter.
 * 
 * Scroll down to paragraph 'Profanity variations' to learn
 * more about how each string creates even more variations.
 */
const variations = [ // 4 * 3 * 3 = 36 variations
  "suckcock", "suckballs", "sucknuts", "suckmycock",
  "suckmyballs", "suckmynuts", "suckmacock", "suckmaballs",
  "suckmanuts", "sugcock", "sugballs", "sugnuts",
  "sugmycock", "sugmyballs", "sugmynuts", "sugmacock",
  "sugmaballs", "sugmanuts", "lickcock", "lickballs",
  "licknuts", "lickmycock", "lickmyballs", "lickmynuts",
  "lickmacock", "lickmaballs", "lickmanuts", "ligcock",
  "ligballs", "lignuts", "ligmycock", "ligmyballs",
  "ligmynuts", "ligmacock", "ligmaballs", "ligmanuts",
];
```

### Profanity variations

The script creates variations of all strings added to the filters, except for the whitelist.
The block of code is simplified to better explain how filter variations are created.
```Javascript
/**
 * `filters.original` contains the exact strings that you added to the filters.
 * 
 * `filters.noNumbers` changes all numbers to their linked letter,
 * and all other filters are based on `filter.noNumbers`.
 * 
 * Number-letter linked: 1→i, 2→z, 3→e, 4→A, 5→s, 6→b, 7→T, 8→B, 9→g
 */
const filters = {
  original:       [ "p3nis", "www.bit", "b1tch", "z00m meeting", "thicccccc girl" ],
  noNumbers:      [ "penis", "www.bit", "bitch", "zoom meeting", "thicccccc girl" ],
};

/**
 * This is an example message that is split on its spaces.
 * The script creates variations (slices) by joining words together.
 */
const message = "You love pen111111s everywhere.";
const slices = [
  "You", "love", "pen111111s", "everywhere.",
  "You love", "love pen111111s", "pen111111s everywhere.",
  "You love pen111111s", "love pen111111s everywhere.",
  "You love pen111111s everywhere."
];

/**
 * For this example we take `slices[2]`.
 * The string is modified the same way that the filters are modified.
 * Each modified string is searched in the corresponding filter.
 */
const slice = "pen1s";
// ... more string modifications

/**
 * Object `checks` contains properties with a boolean.
 * If at least one boolean is true, then it means we have found profanity.
 */
const checks = {
  original: filters.original.includes(slice),             // "pen1s" → false
  noNumbers: filters.noNumbers.includes(sliceNoNumbers),  // "penis" → true
};
```

### Mod actions

The script has 3 moderation actions.<br/>
• `NOTIFY` posts a moderation log without kicking or banning the player.<br/>
• `KICK` kicks the player and posts a moderation kick log.<br/>
• `SOFT_BAN` bans the player and posts a moderation ban log.<br/>
