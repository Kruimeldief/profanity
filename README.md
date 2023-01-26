# Profanity filter

File [`filters.json`](https://github.com/Kruimeldief/profanity/blob/main/filters.json) contains a JSON with different filter types.<br/>
The moderation script fetched this file every 5 minutes to keep its filter up-to-date.

### Word filter

JSON property `wordFilter` has profane words that are used to filter bad language in both player messages and names.

### Name filter

JSON property `nameFilter` has profane words that are used to filter bad language in player names only.<br/>
Player messages are excluded because this filter is meant to put additional restrictions on player names only.<br/>
Example: a player may say 'gay' in the chat but may not use 'gay' in its name.

### Whitelist

JSON property `whitelist` has whitelisted words.<br/>
These words are applied to all profanity checks (player message, name and chat history).<br/>
If any part of the string marked as profanity is part of the whitelist, then the whole string is no longer profane.

### Sentence filter

JSON property `sentenceFilter` has an array with objects that facilitate variations of word sentences.<br/>
`filter`: select a filter to which the profanity variations are added: wordFilter or nameFilter.<br/>
`modAction`: select a moderation action to specify how a player should be handled: KICK or SOFT_BAN. (case sensitive)<br/>
`list`: A nested array where each word in a nested array is added to all strings in the next nested array.<br/>
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
    ["suck", "sug", "lick", "lig"],
    ["", "my", "ma"],
    ["cock", "balls", "nuts"]
  ]
};

/**
 * The moderation script creates and adds the
 * following profanity strings to the nameFilter.
 * 
 * Scroll down to paragraph 'Profanity variations' to learn
 * more about how each string creates even more variations.
 */
const stringsToAdd = [ // 4 * 3 * 3 = 36 variations
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

The moderation script will create variations of words you add to any filter, except for the whitelist.<br/>
This block of code is simplified for the purpose of explaining which variation filters are created from strings you add to filters.
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
const message = "You love pen111111s everywhere."
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
}
```

### Mod actions

The moderation script uses 5 moderation actions: `KICK`, ~~`KICK_REVIEW`~~, `SOFT_BAN`, ~~`SOFT_BAN_REVIEW`~~, and ~~`HARD_BAN`~~.<br/>
The strikken ModActions are depreciated and are automatically converted to either a kick or soft ban.<br/>
`KICK`:            words in this category cause the script to kick the player.<br/>
`KICK_REVIEW`:     words in this category cause the script to kick the player.<br/>
`SOFT_BAN`:        words in this category cause the script to soft ban the player.<br/>
`SOFT_BAN_REVIEW`: words in this category cause the script to soft ban the player.<br/>
`HARD_BAN`:        words in this category cause the script to soft ban the player.
