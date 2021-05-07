qBittorrent is currently translated into approximately ~70 languages and we are always looking for new translators to help us make qBittorrent available into as many languages as possible.

# Translating strings used in NSIS installer
You can find the files at: https://github.com/qbittorrent/qBittorrent/tree/master/dist/windows/installer-translations

Feel free to submit a Pull Request (PR) for the changes you made.

# Translating qBittorrent
We are using Transifex to localize qBittorrent. For the localization progress, visit the [project page](https://www.transifex.com/sledgehammer999/qbittorrent/).

If you want to help, sign up for free, select the language you want to translate and request to join the translation team.
Once you are accepted as a member of the team, you can proceed with the translation.

Transifex provides an easy to use web interface to work on the translation.

You can alternatively work locally using tools such as **Qt Linguist** and submit your changes using the Transifex web interface or the [command-line client](https://docs.transifex.com/client/introduction). Before pushing changes make sure your local files are updated to latest or you might override the work of others.

## Things you need to pay attention to
* HTML tags<br>
We tried not to put much html code in the translatable strings but some of them still contain some HTML tags in them. If it is the case, please recopy the html tags in the translation *as is* and translate the rest of the string. The HTML **MUST** be present in the translation and at the same location.

* `%1`, `%2`, `%3`...<br>
These were introduced to give the translator more flexibility. They must appear *as is* in the translated string but their location *can* change (this is the point). Some languages require that some words are moved in the sentence in order to be understandable. (i.e.: `Finished to download %1.` can be translated in french as `Le téléchargement de %1 est terminé.`)<br>
![Percentages.png](https://www.qbittorrent.org/wiki-images/Percentages.png)

* Keyboard shortcuts: `&`<br>
They should be present in the translation too. Just recopy them at the same location.

* Comments<br>
Sometimes, it may be important to understand to context in order to translate well. This is why I sometimes include comments in the translation files. Refer to *Context* in *More details* on the online editor.

# Translating Bittorrent specific terms
If you need help understanding some Bittorrent specific terms, you can refer to these pages:
* [Wikipedia - Glossary of BitTorrent terms](https://en.wikipedia.org/wiki/Glossary_of_BitTorrent_terms)
* [BTFaq - Bittorrent terms](https://web.archive.org/web/20101120190033/http://www.dessent.net/btfaq/#terms)
