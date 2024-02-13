# cylc-textmate-grammar

![test](https://github.com/cylc/cylc-textmate-grammar/workflows/test/badge.svg?branch=master&event=push)

This repository provides a TextMate grammar for Cylc workflow configuration (`suite.rc` and `.cylc`) files, enabling syntax highlighting and other features.

`cylc.tmLanguage.json` is the grammar file used by plugins for editors:
- VSCode - [cylc/vscode-cylc](https://github.com/cylc/vscode-cylc)
- Atom - [cylc/language-cylc](https://github.com/cylc/language-cylc)

It is also used to build a TextMate bundle at [cylc/Cylc.tmLanguage](https://github.com/cylc/Cylc.tmbundle/) which is compatible with:
- TextMate
- PyCharm
- WebStorm
- Sublime Text 3

## Usage

If you're using an editor listed above, follow the link next to it for instructions. If not, check [cylc/cylc-flow/issues/2752](https://github.com/cylc/cylc-flow/issues/2752) for support.

## Copyright and Terms of Use

Copyright (C) 2008-<span actions:bind='current-year'>2024</span> NIWA & British Crown (Met Office) & Contributors.

[![License](https://img.shields.io/github/license/cylc/cylc-textmate-grammar.svg?color=lightgrey)](https://github.com/cylc/cylc-textmate-grammar/blob/master/LICENSE)

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## Contributing

**Note:** the `cylc.tmLanguage.json` file is generated from the JavaScript file(s) in `src/`. Contributors should edit the JavaScript files, **not** the JSON file; the JSON file should only be generated using the build script (using [node](https://nodejs.org/en/)).

### Getting started

If you are new to TextMate grammars, there are a number of resources to look at:
- [TextMate manual - language grammars](https://macromates.com/manual/en/language_grammars)
- [VSCode syntax highlight guide](https://code.visualstudio.com/api/language-extensions/syntax-highlight-guide)
- [Atom flight manual - creating a legacy TextMate grammar](https://flight-manual.atom.io/hacking-atom/sections/creating-a-legacy-textmate-grammar/)

### Modifying the grammar

As noted above, instead of editing the `cylc.tmLanguage.json` file directly (as is done in the VSCode guide, for example) you should edit `src/cylc.tmLanguage.js` instead.

Each distinct tmLanguage pattern should be represented by a class, whose constructor sets a property called `pattern` that is an object, e.g.:
```javascript
class LineComment {
    constructor() {
        this.pattern = {
            name: 'comment.line.cylc',
            match: '(#).*',
            captures: {
                1: {name: 'punctuation.definition.comment.cylc'}
            }
        };
    }
}
```
or a property called `patterns` that is an array, e.g.:
```javascript
class GraphSyntax {
    constructor() {
        this.patterns = [
            {include: '#comments'},
            new Task().pattern,
            {
                name: 'keyword.control.trigger.cylc',
                match: '=>'
            },
    // etc.
```
Notice above the inclusion of the class `Task`'s pattern object.

Remember that the regex escape character `\` needs to be itself escaped. E.g., for the regex `\s` you need to write `\\s`.

The `exports.tmLanguage` object is where the collection of patterns should go, i.e.:
```javascript
exports.tmLanguage = {
    scopeName: 'source.cylc',
    name: 'cylc',
    patterns: [
        {include: '#comments'},
    ]
    repository: {
        comments: {
           patterns: [
               new LineComment().pattern,
           ]
        },
        graphSyntax: {
            patterns: [
                ...new GraphSyntax().patterns,
            ]
        },
    // etc.
```
where the [spread syntax operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) (3 dots) is used to expand the `GraphSyntax().patterns` array.

### Build

To generate the JSON file, you will need [NodeJS](https://nodejs.org/en/). After you've made changes to the js grammar file, generate the JSON file using
```shell
npm run build
```

The compiled result of the examples above would be the following `cylc.tmLanguage.json`:
```JSON
{
    "scopeName": "source.cylc",
    "name": "cylc",
    "patterns": [
        {"include": "#comments"}
    ],
    "repository": {
        "comments": {
            "patterns": [
                {
                    "name": "comment.line.cylc",
                    "match": "(#).*",
                    "captures": {
                        "1": {"name": "punctuation.definition.comment.cylc"}
                    }
                }
            ]
        },
        "graphSyntax": {
            "patterns": [
                {"include": "#comments"},
                {
                    "name": "meta.variable.task.cylc",
                    "match": "\\b\\w[\\w\\+\\-@%]*"
                },
                {
                    "name": "keyword.control.trigger.cylc",
                    "match": "=>"
                },
            ]
        }
    }
}
```

### Testing

(Note: if you're unable to run the tests locally, GitHub Actions are set up to automatically run the tests on pull requests.)

If you haven't already installed the development dependencies, run
```
npm install
```

#### Style tests
To run ESLint for style testing JavaScript files:
```
npm run lint
```

#### Unit tests
We're using [vscode-tmgrammar-test](https://github.com/PanAeon/vscode-tmgrammar-test) for testing the textmate grammar. Test files reside in the `/tests` directory. These are essentially Cylc workflow files that are annotated with comments. The comments detail what the expected scopes of the test line are. The comments are read in by vscode-tmgrammar-test and compared to the actual applied scopes. An example test might be:
```ini
    foo = bar
#   ^^^       variable.other.key.cylc
#       ^     keyword.operator.assignment.cylc
#         ^^^ meta.value.cylc string.unquoted.value.cylc
#   ^^^^^^^^^ meta.setting.cylc
```
For docs, follow the link to vscode-tmgrammar-test above.

To run the unit tests:
```
npm test
```
