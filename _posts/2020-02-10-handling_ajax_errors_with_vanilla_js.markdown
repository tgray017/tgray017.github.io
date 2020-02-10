---
layout: post
title:      "Handling AJAX Errors with Vanilla JS"
date:       2020-02-10 18:35:39 +0000
permalink:  handling_ajax_errors_with_vanilla_js
---


For my Front-End Web Programming final project, I decided to create a word game. The basic premise of the game is that the user is given a rule with which to follow, and tries to guess as many words as they can that follow that rule (e.g. words that start with "s" and end with "w"). In order to do this, I created both a Rails-backed API in which to store the `games`, `rounds`, and `words` that were guessed by the user, as well as the frontend that the user interacts with. Throughout the three rounds of the game, the user will try to guess as many words that follow the rule in order to gain as many points as possible. Once a user submits a word, the Rails backend performs some internal checks to ensure the word follows the rule, and calls the [Merriam Webster](https://dictionaryapi.com/) API to make sure the word actually exists in the English language. When a user guesses a valid word, that word appears on the screen along with its point value, which is calculated based on the rules of [Scrabble](https://scrabble.hasbro.com/en-us/faq).

You may be asking, "what happens when a user guesses an invalid word?" Well, that's where error handling in AJAX comes into play. I could have simply chosen to throw away the errors entirely so that nothing was communicated to the user when they submitted an invalid word, but I figured if the user is confident that the word they're guessing is a real word, they could think there was something wrong with the app when the word doesn't display on the screen and as a result, submit the same word over and over again and become frustrated, potentially quitting the game in protest. Therefore, I needed to figure out a way to communicate to the user that the word that they submitted was not valid, without being too intrusive. One option would be to flash the error message on the screen, but then the user would spend precious seconds reading the error message instead of guessing more words. I really just needed a way to inform the user quickly that their word was invalid.

I chose to communicatate this in two ways:
1. If the word they had guessed either didn't exist in the Merriam Webster API or didn't follow the rule, the screen would flash red for a split second.
2. If they had already guessed the word and it was valid, the screen would flash orange for a split second.

This would be a quick and intuitive way to get the message across to the user that the word they had guessed was invalid.

I was able to implement this using vanilla JS with just a few different functions and some simple CSS styling. First, I defined a `createWord()` function in my adapter that would communicate with the Rails API using `fetch`. If the JSON response from the server included errors, it would throw a generic error using the [throw](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw) keyword, otherwise it would return the JSON response itself:

```
// frontend/src/adapters/WordsAdapter.js

createWord(word) {
  const configurationObject = {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Accept": "application/json"
    },
    body: JSON.stringify({
      "word": word
    })
  }

  return fetch(this.baseUrl, configurationObject)
  .then(resp => {
    return resp.json()
  })
  .then(json => {
    if (json.errors) {
      throw new Error (json.errors)
    } else {
      return json
    }
  })
}
```

The `createWord()` function above, which is defined in `frontend/src/adapters/WordsAdapter.js`, is called from within the `createWord()` function defined in `frontend/src/components/words.js`. If the return value of the function above is not an error, the result will feed into the `then` statement below; if it's the generic error we threw above, the result will feed into the `catch` statement below the `then`. Within the `catch` we have some logic to flash the screen orange if the error message (defined by the Rails API) indicates that the user is trying to guess a word they've already guessed, otherwise the screen will flash red. This was as simple as adding a couple classes to the `body` and `main` HTML elements in order to create the coloring and opacity of the flash, and removing those classes a split second later using [setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout):

```
// frontend/src/components/words.js

createWord(word) {
  this.newWordContent.value = ''
  this.adapter.createWord(word.toLowerCase())
  .then(word => {
    this.words.push(new Word(word))
    this.displayWords()
  })
  .catch(err => {
    if (err.message === "Word must be unique for the current round") {
      this.flashScreen('orange')
    } else {
      this.flashScreen('red')
    }
    console.log(err.message)
  }
)}

flashScreen(color) {
  this.addFlash(color)
  setTimeout(this.removeFlash, 100, color)
}

addFlash(color) {
  let body = document.getElementsByTagName("body")[0]
  let main = document.getElementsByTagName("main")[0]

  body.setAttribute("class", `flash-${color}`)
  main.setAttribute("class", "move-back")
}

removeFlash(color) {
  let body = document.getElementsByTagName("body")[0]
  let main = document.getElementsByTagName("main")[0]

  body.classList.remove(`flash-${color}`)
  main.classList.remove("move-back")
}
```

Within `frontend/style.css`:
```
.move-back {
  opacity: 0.5;
}

.flash-red {
  background-color: rgb(255, 96, 112);
}

.flash-orange {
  background-color: rgb(255, 122, 73);
}
```

There you have it--a simple technique to differentiate between errors and communicate the error to the user in an unobtrusive way. Of course, there is much more that can be done with error handling in JavaScript, and anyone who is looking for more information on the subject should take a look at MDN's documentation on [exception handling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling#Exception_handling_statements) statements.

For anyone interested in seeing the error handling in action, check out my [Scribbler](https://github.com/tgray017/scribbler) app!

