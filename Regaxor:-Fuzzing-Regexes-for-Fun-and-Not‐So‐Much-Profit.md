## Backstory

#### Note: If you don't like bug bounty drama, feel free to skip this part.

It all started with the public disclosure of a [HackerOne report](https://hackerone.com/reports/307672) submitted to [Keybase](https://hackerone.com/keybase) by another researcher. It was a minor character escaping issue, but one that probably had the highest impact of all vulnerability reports submitted to their program—given that it was issued their highest bounty payout at the time. I immediately noticed the "[patch](https://github.com/keybase/client/pull/10277/files)" they implemented was insufficient, and it was only a matter of minutes before I submitted a new vulnerability report (as quickly and as fast as possible). Later, they received 4 duplicate reports of the same issue. But I'm pretty sure mine was first despite their total lack of transparency [1].

Long story short, they handled my report in a totally unprofessional manner to the point that it was my worst experience participating in a bug bounty program. They even went as far as to reopen my report and mark it as a duplicate of another that was submitted after mine, rewarding it too, just because I later criticized the way they handled this—very politely and in passive language. They wouldn't agree to disclose my report, so I had to export it externally for those who would like to [take a look](https://drive.google.com/file/d/1ZPZsnxEHIXGZ5rGv1cxq3rLDLZD7GO3o/view). Not cool, not cool at all. The only good thing that came out of this mess was the regex fuzzing tool that I'm going to talk about here.

## Regaxor (RegExp Haxxor)

Whatever you're coding, regular expressions come in handy in various situations and are often very useful but can also be very tricky to get right. Writing a regex that matches what you expect is easy; writing a regex that ___only___ matches what you expect is virtually impossible (except in trivial cases). That's where this tool comes into play—by fuzzing regular expressions, we can easily detect any issues/gotchas before learning about them the hard way.


### Regex Gotchas

Regular expressions can be quite confusing at times. Given an unexpected input, the regex that was working just fine can lead to the weirdest of results.

Some of these gotchas are (NVM the funny titles):

1. In the beginning was the Word
```javascript
let badRegex = /https?:\/\/example\.com\/[\w]*/;
let str = 'Word\nhttps://example.com/';
str.match(badRegex);
// Output: ["https://example.com/", index: 5, input: "Word↵https://example.com/", groups: undefined]

let goodRegex = /^https?:\/\/example\.com\/[\w]*/;
str.match(goodRegex);
// Output: null

'https://example.com/'.match(goodRegex);
// Output: ["https://example.com/", index: 0, input: "https://example.com/", groups: undefined]

```

2. Catch 22
```javascript
let badRegex = /[123]|22/g;
badRegex.exec('22');
// Output: ["2", index: 0, input: "22", groups: undefined]

let goodRegex = /22|[123]/g;
goodRegex.exec('22');
// Output: ["22", index: 0, input: "22", groups: undefined]

```

3. One sneaky dot
```javascript
let str = 'https://exampleXcom';
let badRegex = /^\w+:\/\/example.com$/;
badRegex.exec(str);
// Output: ["https://exampleXcom", index: 0, input: "https://exampleXcom", groups: undefined]

let goodRegex = /^\w+:\/\/example\.com$/;
goodRegex.exec(str);
// Output: null

goodRegex.exec('https://example.com');
// Output: ["https://example.com", index: 0, input: "https://example.com", groups: undefined]

```

4. All or nothing
```javascript
let badRegex = /^\.*|\d+$/g;
'abc'.match(badRegex);
// Output: [""]

let goodRegex = /^[\d.]+$/g;
'abc'.match(goodRegex);
// Output: null

'12.3'.match(goodRegex);
// Output: ["12.3"]

```

5. The word boundary trap
```javascript
let badRegex = /word/;
badRegex.exec('aworda');
// Output: ["word", index: 1, input: "aworda", groups: undefined]

let goodRegex = /\bword\b/;
goodRegex.exec('aworda');
// Output: null

goodRegex.exec('a word');
// Output: ["word", index: 2, input: "a word", groups: undefined]

```

6. Multiline confusion
```javascript
let badRegex = /a.*b/;
badRegex.exec('a\nb');
// Output: null

let alsoBadRegex = /a.*b/m;
alsoBadRegex.exec('a\nb');
// Output: null

let goodRegex = /a[^]*b/;
goodRegex.exec('a\nb');
// Output: ["a↵b", index: 0, input: "a↵b", groups: undefined]

```

7. One escape is not enough
```javascript
let badRegex = 'x\.com';
new RegExp(badRegex).exec('xycom');
// Output: ["xycom", index: 0, input: "xycom", groups: undefined]

let goodRegex = 'x\\.com';
new RegExp(goodRegex).exec('xycom');
// Output: null

new RegExp(goodRegex).exec('x.com');
// Output: ["x.com", index: 0, input: "x.com", groups: undefined]

```

8. Escaping the escaping
```javascript
let str = 'double\\"quotes"';

// Bad.
str.replace(/"/g, '\\"');
// Output: "double\\"quotes\""

// Not bad but not recommended.
str.replace(/(\\|")/g, '\\$1');
// Output: "double\\\"quotes\""

// Better.
str.replace(/\\/g, '\\\\').replace(/"/g, '\\"');
// Output: "double\\\"quotes\""

```

9. Too greedy
```javascript
let badRegex = /<.+><\/.+>/g;
let tags = '<tag attribute="foo"></tag><tag id="foo"></tag>';
badRegex.exec(tags);
// Output: ["<tag attribute="foo"></tag><tag id="foo"></tag>", index: 0, input: "<tag attribute="foo"></tag><tag id="foo"></tag>", groups: undefined]

let notBadRegex = /<.+?><\/.+?>/g;
notBadRegex.exec(tags);
// Output: ["<tag attribute="foo"></tag>", index: 0, input: "<tag attribute="foo"></tag><tag id="foo"></tag>", groups: undefined]

notBadRegex.exec(tags);
// Output: ["<tag id="foo"></tag>", index: 27, input: "<tag attribute="foo"></tag><tag id="foo"></tag>", groups: undefined]

```

10. The misplaced hyphen
```javascript
let badRegex = /[\w -$]+/;
'#'.match(badRegex);
// Output: ["#", index: 0, input: "#", groups: undefined]

let goodRegex = /[\w $-]+/;
'#'.match(goodRegex);
// Output: null

'$100 USD'.match(goodRegex);
// Output: ["$100 USD", index: 0, input: "$100 USD", groups: undefined]

```

At times, writing a regex can feel like walking in a minefield. At other times, regular expressions are the wrong answer—or as Jamie Zawinski puts it `Some people, when confronted with a problem, think "I know, I'll use regular expressions." Now they have two problems.`. So, especially in security-sensitive contexts, you're probably better off not using regular expressions unless you really have to....


#### You can find an online demo at the following URL:
 https://0xsobky.github.io/Regaxor

The following is the core module of the tool (ES6):
```javascript
/**
 * A wrapper object for helper functions.
 */
let regaxorHelpers = {
    padLeft(str) {
        return str[0] + str;
    },

    padRight(str) {
        return str + str[str.length - 1];
    },

    replaceDots(str) {
        return str.replace(/\./g, 'X');
    },

    doubleEscape(str) {
        return str.replace(/\\/g, '\\\\');
    },

    prefixLn(str) {
        return '\r\n\n\r' + str;
    },

    postfixLn(str) {
        return str + '\r\n\n\r';
    },

    replicateLn(str) {
        return str.repeat(2) + '\u2028' + str;
    },

    sliceRight(str) {
        let length = str.length / 2;
        return str.slice(length);
    },

    sliceLeft(str) {
        let length = str.length / 2;
        return str.slice(0, length);
    },

    flip(str) {
        let length = str.length / 2;
        return str.slice(length) + str.slice(0, length);
    },

    reverse(str) {
        str = str.split('');
        str = str.reverse();
        return str.join('');
    },

    changeCase(str) {
        if (str.toLowerCase() === str)
            str = str.toUpperCase();
        else
            str = str.toLowerCase();
        return str;
    },

    shuffle(str) {
        let arr = str.split('');
        let index = arr.length;
        while (index--) {
            let rand = Math.floor(Math.random() * (index + 1));
            [arr[rand], arr[index]] = [arr[index], arr[rand]];
        }
        return arr.join('');
    },

    shift(str) {
        let arr = str.split('');
        let index = arr.length;
        let rand = Math.ceil(Math.random() * 5);
        while (index--) {
            let char = arr[index];
            arr[index] = String.fromCharCode(
                char.charCodeAt(char) << rand
            );
        }
        return arr.join('');
    }
};


/**
 * Fuzz a given regular expression.
 * @param {string} str - A raw string.
 * @param {object|string} re - A regex literal/string literal.
 * @param {boolean} literalFlag - `true` for regex literals.
 * @return {object} array - An array of object literals.
 */
let regaxor = (str, re, literalFlag) => {
    let flags = '';
    let outputs = {'matches': [], 'mismatches': []};
    if (!literalFlag) {
        re = re.trim();
        if (re.startsWith('/') && /\/\w*$/.test(re)) {
            let parts = re.split('/');
            flags = parts[2];
            re = parts[1];
        }
    }
    for (let func of Object.values(regaxorHelpers)) {
        let result;
        let matchStr = func(str);
        if (matchStr === str)
            continue;
        re = flags ? new RegExp(re, flags) : new RegExp(re);
        do {
            let dict = {};
            result = re.exec(matchStr);
            if (result !== null) {
                dict.index = result.index;
                dict.match = result[0];
                dict.input = result.input;
                outputs.matches.push(dict);
            } else {
                outputs.mismatches.push(matchStr);
            }
        } while (re.lastIndex && result !== null &&
            re.lastIndex !== matchStr.lastIndexOf(result) + 1);
    }
    return outputs;
};

```

There's also a dedicated GitHub repository at: [https://github.com/0xSobky/Regaxor](https://github.com/0xSobky/Regaxor).

Feel free to contribute in any way.

Here's a screenshot of the tool in action:
[![screenshot.png](https://github.com/0xSobky/Regaxor/raw/master/data/images/screenshot.png)](https://github.com/0xSobky/Regaxor/raw/master/data/images/screenshot.png)

As the code is already documented, and the UI is simple enough, I guess there's no need for any further explanation. So thanks for reading, and until next time!


## Footnotes

1. Report IDs on [HackerOne](https://hackerone.com/) are sequential and global, meaning that a report with the ID "309567" precedes a report with the ID "309576". Typically, when a report is a duplicate of another, the researcher who submitted the duplicate report is invited to participate in the original one; timestamps do not lie.
