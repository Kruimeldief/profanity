# Profanity filter

File [filters.json](https://github.com/Kruimeldief/profanity/blob/main/filters.json) has different filter types.
The moderation script fetches this file every 5 minutes to keep its filter up-to-date.

### Word filter

The wordFilter is applied to both player messages and names.

### Name filter

The nameFilter is applied to player names only.
Example: a player may say 'I am gay' in the chat but may not use 'gay' in its name.

### Whitelist

Whitelisted words prevent false positives when finding profanity.
If a profane part of a string contains a whitelisted word, that profane part is ignored.
Read paragraph [profanity variations](https://github.com/Kruimeldief/profanity#profanity-variations) for more information about how profanity and whitelisted words are found.
```Javascript
const whitelist = [ "title", "titles", "it", "its", "tilt" ];
const message = "Show me your t its";

/**
 * Read paragraph 'Profanity variations' to learn
 * more about how profanity is found in strings.
 *
 * Checking for whitelisted words uses the same
 * algorithm as explained in 'Profanity variations'.
 */
const foundProfanity = "t its";
const sliceList = [ "t", "its" ];

for (const slice of sliceList) {
  if (whitelist.includes(slice) {
    // "its" is whitelisted so profanity "t its" is ignored.
  }
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
  "suck cock", "suck balls", "suck nuts", "suck my cock",
  "suck my balls", "suck my nuts", "suck ma cock", "suck ma balls",
  "suck ma nuts", "sug cock", "sug balls", "sug nuts",
  "sug my cock", "sug my balls", "sug my nuts", "sug ma cock",
  "sug ma balls", "sug ma nuts", "lick cock", "lick balls",
  "lick nuts", "lick my cock", "lick my balls", "lick my nuts",
  "lick ma cock", "lick ma balls", "lick ma nuts", "lig cock",
  "lig balls", "lig nuts", "lig my cock", "lig my balls",
  "lig my nuts", "lig ma cock", "lig ma balls", "lig ma nuts",
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
  original:       [ "p3nis", "www.bit", "b1tch", "z00m meeting", "thicccccc girl" ], // original
  noNumbers:      [ "penis", "www.bit", "bitch", "zoom meeting", "thicccccc girl" ], // based on original
  noDuplicates:   [ "penis", "w.bit",   "bitch", "zom meting",   "thic girl"      ], // based on noNumbers
  noTriplicates:  [ "penis", "ww.bit",  "bitch", "zoom meeting", "thicc girl"     ], // based on noNumbers
  noSpaces:       [ "penis", "www.bit", "bitch", "zoommeeting",  "thiccccccgirl"  ], // based on noNumbers
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
const slice = "pen111111s";
// ... more string modifications

/**
 * Object `checks` contains properties with a boolean.
 * If at least one boolean is true, then it means we have found profanity.
 */
const checks = {
  original: filters.original.includes(slice),                         // "pen111111s" → false
  noNumbers: filters.noNumbers.includes(sliceNoNumbers),              // "peniiiiiis" → false
  noDuplicates: filters.noDuplicates.includes(sliceNoDuplicates),     // "penis"      → true
  noTriplicates: filters.noTriplicates.includes(sliceNoTriplicates),  // "peniis"     → false
  noSpaces: filters.noSpaces.includes(sliceNoSpaces),                 // "peniiiiiis" → false
};
```

### Mod actions

The script uses 5 moderation actions of which only 2 are still used.
Any depreciated actions convert to either kick or soft ban.<br/>
• `KICK` kicks the player.<br/>
• ~~`KICK_REVIEW`~~ kicks the player.<br/>
• `SOFT_BAN` bans the player.<br/>
• ~~`SOFT_BAN_REVIEW`~~ bans the player.<br/>
• ~~`HARD_BAN`~~ bans the player.
