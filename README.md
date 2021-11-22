# Suggestion for Language "Pattern Folding" Rules
This is a proposed addition to Visual Studio Code to add parameters to configure code folding.

Nothing here is intended to be final. It is a work in progress. If it has a future that may include changes small or huge. Suggestions and feedback would be very welcome.

[Click here](.\patternfolding\introduction.html) for a description of the parameters and proposed functionality.

[This repository](https://github.com/PieterBranderhorst/vscode) is a fork of the vscode source which contains changes to implement this feature.

The code for this is [in this file](https://github.com/PieterBranderhorst/vscode/blob/pattern_folding/src/vs/editor/contrib/folding/patternRangeProvider.ts). Changes to other source files are minor to support loading the folding parameters and to invoke the new code.

Two new editor settings are present. They may not exist in the long term. They are present for now to make it easy to compare two ways of doing things and to find out whether both are wanted:
* `foldingShowEndLine`. Set this to true to keep the last line of folded ranges visible. The default is false, to hide the last line unless it contains some non-comment after the text which ends the fold range.
* `foldingShowTrailingBlank`. Set this to false to hide all blank lines which follow a folded range. The default is true, to keep one blank line visible if there are blank lines following the end of a fold range.
