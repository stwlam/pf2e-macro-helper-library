# PF2E Macro & Helper Library
This module exists because I'm not good enough at TypeScript to contribute directly to Workbench. 
It is a collection of macros and helper functions I've written for PF2e. It will (hopefully) be continually expanding. If you have a macro you'd like to contribute, and it's unsuitable for inclusion in Symon's repo (ie, it uses some of the helper functions provided by this module), feel free to open a PR, or open an issue if you don't want to git.

## Existing Macros:
Macros are accessed via `game.pf2emhl.macros.`
#### Fascinating Performance (`async fascinatingPerformance()`)
Requires one token selected, and at least one target. Has handling for target limits depending on Performance rank, and will ignore any targets with an effect that contains both "Immun" and "Fascinating Performance" in its name, case-**in**sensitive. TODO: Build immunity effect, apply as appropriate (existing behaviour is a holdover from standalone macro)
#### Lashing Currents (`async lashingCurrents()`)
Implements the Relic Gift of the same name. Prompts the user to select a weapon, and adds a Strike RE and attendant WeaponPotency, Striking, and AdjustStrike rules to mirror the weapon's property runes. Run again on the same actor to remove the rules. Does not prevent you from using the weapon's base strike while active; No practical way to do that without being more destructive.
#### Recover Old Lashing Currents (`async recoverOldLashingCurrents()`)
For recovering weapons hidden in flags by the old Lashing Currents macro originally shipped with the PF2e Relics module.
#### Drop Held Torch (`async dropHeldTorch()`) *Requires Item Piles*  
Requires one token only selected, and a currently held torch. Creates an Item Pile containing the torch, removing it from the actor. If the torch was lit, apply that light to the resulting pile token. Significant generalization and improvements planned.

## Existing Helper Functions:
Helpers are accessed via `game.pf2emhl.`

---
### Token and Target helpers  
These are mostly replacing one-liners, but I got tired of typing out all my error handling for this stuff every time.
#### `oneTokenOnly(fallback = true)`  
Returns a single token placeable, or errors if more than one. If `fallback` is `true` (default), will attempt to find a token of the user's assigned character on the current scene if no others selected.
#### `anyTokens(fallback = true)`  
Returns an array of selected token placeables, erroring if none selected, unless `fallback` is `true`, as above.
#### `oneTargetOnly(useFirst = false, user = game.user)`  
Returns a single token placeable, targetted by the specified user (defaulting to the user running the macro). Errors if user has no targets, or more than one target. Can override the latter behaviour with the `useFirst` parameter, which will suppress errors for >1 target and simply return the first target of the given user.
#### `anyTargets(user = game.user)`  
Returns an array of token placeables, targetted by the specified user, as above. Errors if no targets found.

---
### String helpers
#### `localize(str, data = {}, { defaultEmpty = true } = {})`  
A reimplementaion of `game.i18n.format()` with some extra handling: 
- Localization strings which contain curly braces that are *not* intended to be substitutions are supported via escaping the opening brace (`\{`)
- Substitutions that are not provided in `data` but exist in the localization string will default to an empty string (instead of `undefined`) if `defaultEmpty` is `true`
#### `prependIndefiniteArticle(string)`  
Returns the provided string prepended with either 'a' or 'an' (lowercase), as appropriate.
#### `capitalize(string)`  
Returns the provided string with the first character having `.toUpperCase()` applied to it.

---
### Error Handling helpers
#### `localizedError(str, data = {}, { notify = null, prefix = "", log = {} } = {})`  
Returns an `Error` with the message having been passed through `localize()` as above, and with `prefix` prepended to it. if `log` is provided and is an object, that whole object is passed to `console.error()` for ease of debugging. `notify` determines whether or not to produce a banner notification in addition to the console error, and if left nullish will be dictated by the module setting for banner notifications. 
#### `localizedBanner(str, data = {}, { notify = null, prefix = "", log = {}, type = "info", console = true} = {})`  
Localizes `str` with `data`, preprends `prefix`, and calls `ui.notifications[type]` with the result and `console`. Errors if `type` is not `info`, `warn`, or `error`. If `notify` is nullish, falls back on the module setting, as above. If `log` is provided and is an object, it will be passed to `console[type]()`. `notify` functions as above.
#### `MHLError(str, data = {}, { notify = null, prefix = "MacroHelperLibrary: ", log = {}, func = null } = {})`  
A simple wrapper on `localizedError` above, pre-fills the prefix for this library's calls, and provides the `func` variable which, if provided, is inserted between the prefix and the rest of the error string, for more a more granular 'where did this error come from' report.

---
### PF2e-specific Helpers
#### `levelBasedDC(level)`  
Returns the appropriate value from the level-based DC table, given `level` (a number from -1 to 25). Errors if passed a non-number, and defaults to level 25 if passed a number outside its range.
#### `async setInitiativeStatistic(actor, statistic = "perception")`  
One-liner wrapping an update to `actor.system.initiative.statistic`
#### `async pickItemFromActor(actor, { itemType = null, otherFilter = null, held = false, title = null, dialogOptions = {}, errorIfEmpty = true } = {})`  
Utilizes the `pickAThingDialog` helper documented below to prompt the user to select an item owned by the provided actor, that matches the provided filters. 
- `itemType`: must be either a valid PF2e item type, or `'physical'`, which is also the default.
- `otherFilter`: must be a function that takes one parameter and returns bool (as you would pass to `.filter()`)
- `held`: if true, restrict to items currently held by the actor (any # of hands)
- `title` and `dialogOptions` are passed through to `pickAThingDialog` (see above)
- `errorIfEmpty`: if false, will return null if no items matching the given filters is found, otherwise throws an error.
#### `async getAllFromAllowedPacks({ type = "equipment", fields = [], filter = null, strictSourcing = true, fetch = false } = {})`  
Returns an array of index entries (if `fetch` is `false`) or Documents (if `fetch` is `true`), matching the provided filter, while respecting the Compendium Browser's allowed packs and sources settings.
- `type`: Must be the slug of a compendium browser tab, or one of the allowed aliases: `ability` (aliases: `action`), `bestiary` (aliases: `npc`, `actor`), `campaignFeature`, `equipment`, `feat`, `hazard`, or `spell`.
- `fields`: Any data paths your filter requires, beyond the standard compendium indicies for that document type. 
- `filter`: Must be a function that takes one parameter and returns bool (as you would pass to `.filter()`)
- `strictSourcing`: If `true`, will suppress documents with missing source information, otherwise they're let through. Depending on how thorough your content module author(s) is at setting publication data, this could block a significant number of documents.
- `fetch`: If `true`, returns whole documents, otherwise (default) returns only the compendium index data.
**NOTE**: if the CB has not yet been initialized when called, there will be significant (couple seconds) delay on first run while that's handled.

---
### Others (TODO: better categories)
#### `isOwnedBy(doc, user)`  
Returns bool: Does the provided user have ownership permissions on the provided document? If provided token placeable or document, operates on the associated actor (fallback code stolen from warpgate, thanks honeybadger)
#### `getIDsFromFolder(root)`  
Takes a folder (Document or ID) in `root`, and returns a flattened array of ids from every Document in the folder structure. World folders only.
#### `async applyOwnershipToFolderStructure(root, exemplar)`  
Takes `root` as above, and a Document (or any object with an `ownership` property) in `exemplar`, and applies the latters ownership data to every document in the folder structure. Doesn't handle documents/folders in compendia. Be careful, as once an ownership level has been set for a particular user, it can not be reset to inherit via this method, and will need to be explicitly downgraded if so desired.
#### `async pickAThingDialog({ things = null, title = null, thingType = "Item", dialogOptions = {} } = {})`  
Provides a dialog to pick from a list of choices. Merges provided `dialogOptions` with its defaults (`{ jQuery: false, classes: ["pick-a-thing"] }`). `title` defaults to `Pick a[n] {thingType}` if not provided. `things` must be an array of objects that look like: 
```js
{
  label: <string>,
  value: <string>,
  img?: <string>,
  identifier?: <string>
}
```
Example image (produced via the following helper): 

![](https://i.imgur.com/5pHRAl7.png)

The purple text is the `indentifier`, which can be supplied to disambiguated things with duplicate names. Currently produces a single button per provided `thing`, regardless of thing count. 
**TODO**: implement select menu fallback for > configurable limit of items, improve styling generally.
