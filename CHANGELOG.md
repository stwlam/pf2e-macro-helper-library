# PF2e Macro & Helper Library Changelog

## Version 0.1
- Existance begins.

## Version 0.1.1
- reattempt github actions

## Version 0.1.2
- fix imports

## Version 0.1.3
- fix error string in `oneTokenOnly()`
- actually import targetHelpers
- add some debug to `applyOwnershipToFolderStructure()`
- fixed data path in `setInitiativeStatistic()`

## Version 0.2.0
- add `updateInitiativeSkillsDialog()` macro
- add `MHDialog` class
- various tidyup
- add `isEmpty()` helper
- add `log` and `mhlog` helpers

## Version 0.2.1
- fix changelog

## Version 0.2.2
- switch to SASS for styling
  - add SASS compilation to github actions
- remove `isEmpty` helper, I didn't realize there was already one in `foundry.utils`
- implement new version of updateInitiativeStatistics
- document updateInitiativeStatistics

## Version 0.2.3
- fix imports in macros/index.mjs
- update workflow again?

## Version 0.2.4
- update workflow again again

## Version 0.2.5
- become npm package, write own simple sass compile job
- it works! release finally.

## Version 0.2.6
- Attempt new foundry package release API
- Update MHLDialog's template-as-content support to allow prototype method and property use, matching the behaviour of foundry's `renderTemplate()`

## Version 0.2.7 
- *Actually* update the template call properly.