---
title: Auto Complete and Dictionary
date: 2024-12-25 21:07:00 Z
---

# VIM Built-in Complete

VIM itself has built-in auto completion, doesn't need any plugin. Complete sources come from the
open buffers. Whenever you type in the middle of the word, press CTRL-N will trigger auto completion.
CTRL-N, CTRL-P to select from the candidate word list.

- [Vim's built-in auto complete is all you need](https://www.youtube.com/watch?v=tFD2Ia5TIQ8)

# AUTO Complete Types

- Buffer
- Tags
- File and Path
- Line
- Dictionary
- Spell
- LSP
- RG
- Command

# NVIM Plugins

## [A completion engine plugin for neovim written in Lua](https://github.com/hrsh7th/nvim-cmp)

- Full support for LSP completion related capabilities
- Other complete sources plugins could integrated with the engine within unified appearance

# Spell Check

```vim
:set spell spelllang=en_us

OR in NVIM lua

vim.opt.spell = true
vim.opt.spelllang = { 'en_us' }
```

Once set this, Vim will highlight misspelled words. It also categorizes misspelled words,
highlights rare words, words that are not capitalized (but should be). And even unusual way of
expression in my case, not sure it's built-in or coming from other plugins...

- Red with misspell (in my case)
- Yellow with rare (in my case)
- ]s Move cursor to the next misspelled word
- [s Previous misspelled word
- =z suggestion list to correct the misspell
- zg add a word to spell dictionary
- [spell source for nvim-cmp based on vim's spellsuggest](https://github.com/f3fora/cmp-spell)
- [spellcheck dictionary for programmers](https://github.com/psliwka/vim-dirtytalk)
  - Aid with writing technical documentation (such as project's READMEs, runbooks, code comments,
    etc.), by providing you with a list of commonly used programming-related works
- [Fast Add Spell list reference](https://stackoverflow.com/questions/71625240/how-to-add-all-currently-misspelled-words-to-vims-spell-list)

> [Spell Auto correct plugin for Neovim](https://github.com/ck-zhang/mistake.nvim)
>
> - over 20,000 entries for correction
> - based on GitHub's "Fixed typo" commits

# Dictionary

[OS spell dictionary](https://unix.stackexchange.com/questions/213628/where-do-the-words-in-usr-share-dict-words-come-from)

[WN]()

- WordNet® is a large lexical database of English. Nouns, verbs, adjectives and adverbs are grouped
  into sets of cognitive synonyms (synsets), each expressing a distinct concept. Synsets are
  interlinked by means of conceptual-semantic and lexical relations. The resulting network of
  meaningfully related words and concepts can be navigated with the browser(Link is external).
  WordNet is also freely and publicly available for download. WordNet's structure makes it a useful
  tool for computational linguistics and natural language processing.

[FreeDict](https://freedict.org/)

- FreeDict nowadays provides over 140 dictionaries in about 45 languages and thanks to its members,
  grows continuously

[GCIDE](https://gcide.gnu.org.ua/)

- GNU Collaborative International Dictionary of English
- Free dictionary derived from Webster's Revised Unabridged Dictionary Version published 1913

> NOT FOR FREE
>
> > [Oxford English Dictionary](https://www.oed.com/)
> >
> > - An unsurpassed guide for researchers in any discipline to the meaning, history, and usage of
> >   over
> > - 500,000 words and phrases across the English-speaking world
> >
> > [oxford learners dictionaries](https://www.oxfordlearnersdictionaries.com/)
> >
> > - **Oxford 3000 and Oxford 5000 position paper**

## Webster 1913 Classical

- [Why Webster 1913](https://luxagraf.net/src/how-use-websters-1913-dictionary-linux-edition)
- [Command Line](https://jsomers.net/blog/dictionary)
  - 1
  - 2

[Webster Dictionary Online](https://dictionaryapi.com/)

- Free API (1000 queries per day per API key)

```
https://www.dictionaryapi.com/api/v3/references/learners/json/apple?key=your-api-key
https://dictionaryapi.com/products/json
https://media.merriam-webster.com/audio/prons/[language_code]/[country_code]/[format]/[subdirectory]/[base filename].[format]
```

```json
hwi:{
  hw:"ap*ple",
  prs:[
      {
          ipa:"ˈæpəl",
          sound:{
              audio:"apple001"
          }
      }
  ]
},
```

[Perl script to search Noah Webster's classic 1913 dictionary](https://github.com/dnmfarrell/WebsterSearch)

## IPA/sound

The target is setting up a tool used in nvim to easy access phonetic and sound of word under cursor

### Reference Plugins Wrappers

> Following plugins are outdated, just for personal reference

- [A Vim plugin for phonetics, Listening to phonetics](https://github.com/soywod/phonetics.vim)
- [neovim for Oxford Dictionaries API](https://github.com/matsui54/OxfDictionary.nvim)
- [Python wrapper for Urban Dictionary API](https://github.com/bocong/urbandictionary-py)
- [Pthon wrapper around the Merriam-Webster APIs](https://github.com/pfeyz/merriam-webster-api)

### Python Script example with v3 API

Python `soundplay` need to lock the sound resources, but what I'm doing is to put everything in a
docker, doesn't work with easy install. Just use external utilities to play the sound.

```python

    def send_request(self, word, method="GET"):
        """Make a GET request to the API"""
        full_uri = f"https://www.dictionaryapi.com/api/v3/references/learners/json/{word}?key={self.api_key}"
        response = self.session.request(
            method, full_uri, timeout=self.timeout, headers=self.headers
        )

        return response

    def pronunciation(self, word):

        try:
            res = self.send_request(word).json()
            ipa = res[0]["hwi"]["prs"][0]["ipa"]
            audio_name = res[0]["hwi"]["prs"][0]["sound"]["audio"]
        except Exception as e:
            print("get pronunciation failed:", e)
            traceback.print_exc()
            return

        if len(audio_name) < 2:
            print("audio name parse error!")
            return

        subdir = self.subdirectory(audio_name)

        audio_url = f"https://media.merriam-webster.com/audio/prons/en/us/mp3/{subdir}/{audio_name}.mp3"

        audio_data = requests.get(audio_url).content

        with tempfile.NamedTemporaryFile() as fp:
            fp.write(audio_data)
            fp.seek(0)
            print(ipa)
            os.system(f"paplay {fp.name}")
            fp.close()
```

## Sound Play in Mac OS hosted Container

- [How to](https://stackoverflow.com/questions/40136606/how-to-expose-audio-from-docker-container-to-a-mac)

- [Test Reference](https://gist.github.com/todgru/42bcaa9b38498b266dc07d6bab100e27)

- [Docker sound on MacOS test](https://gist.github.com/seongyongkim/b7d630a03e74c7ab1c6b53473b592712)

- [Enabling Sound Card Access in Docker Containers Using PulseAudio](https://medium.com/@18bhavyasharma/enabling-sound-card-access-in-docker-containers-using-pulseaudio-d52ff1f5eee4)
- [docker sound box on MacOS](https://devops.datenkollektiv.de/running-a-docker-soundbox-on-mac.html)

- [python playsound](https://stackoverflow.com/questions/64414917/namespace-gst-not-available-error-when-using-playsound-module-in-raspbian-os)

- [ALSA/Pulseaudio](https://github.com/mviereck/x11docker/wiki/Container-sound:-ALSA-or-Pulseaudio)

- [PulseAudio Server](https://askubuntu.com/questions/972510/how-to-set-alsa-default-device-to-pulseaudio-sound-server-on-docker)

- [Offline dict with pronunciations](https://askubuntu.com/questions/170775/offline-dictionary-with-pronunciation-and-usages)

  Method in the post is not workable in my case. Change to

  ```
  qq        # Use register q record Macro
  ]szgq     # Go to the next miss spell dectected, add the word into spell list, stop recording
  100@q     # reapt the macro 100 times

  ```

### MacOS Host

```cmd
alias sound='pulseaudio --load=module-native-protocol-tcp --exit-idle-time=-1 --daemon'
brew install -v -d pulseaudio
sound
```

### Docker

- Docker file add package pulse audio
- Docker file add ENV PULSE_SERVER=docker.for.mac.localhost
- Docker compose file Mount host pulse configuration as volume to container - /Users/ospost/.config/pulse:/home/ospost/.config/pulse

## container UID/GID setting for correct privilege

[Containers as a Non-root User with a Custom UID / GID](https://nickjanetakis.com/blog/running-docker-containers-as-a-non-root-user-with-a-custom-uid-and-gid)
[uid and gid in docker container](https://medium.com/@mccode/understanding-how-uid-and-gid-work-in-docker-containers-c37a01d01cf)

# Translation
