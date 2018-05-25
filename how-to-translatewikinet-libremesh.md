* translatewiki.net process
** general
1) Write code, interface created or modified, new strings created.
2) Run script to generate .pot or .json files in libremesh repo, and commit these changes to the appropriate branch:
   - lime-packages = develop
   - lime-app = develop
   - chef = master
3) Patrick gets those changes into the translation-bridge repo by running these commands on his computer (maybe there's a way to automate this with hooks?):
   1. cd ~/Documents/GitHub-translation-bridge/lime-app [or lime-packages or chef]
   2. git status [make sure I'm in the correct branch]
      - lime-packages = feature/tw-translations
      - lime-app = issue-119
      - chef = master
   3. git fetch upstream
   4. git rebase upstream/develop
   5. git push origin
4) translatewiki.net pulls in .pot and .json files from the translation-bridge repo from the appropriate branches (in "git status" above)
5) People translate into new and/or existing languages.
6) translatewiki.net pushes to the appropriate branches in the translatio-bridge repo, every Monday and Thursday.
7) Patrick submits a pull request to the libremesh repo.
8) Someone merges the new translations into the libremesh repo.
9) If there was a new language added, then we add that language to selection menus:
   - lime-packages a.k.a. LuCI interface
     1. file ?
   - lime-app, nothing to do, language is not selectable
   - chef
     1. file chef.html
     2. the lines following this line create the language menu
	- <select class="custom-select" id="lang" name="lang" onchange="translate();">
	- which is currently line 27
     3. Add languages to the menu, copying language name in target language from translatewiki.net, by going to Translating:LibreMesh > Statistics, clicking on the language, and copying its name from the upper right corner of the page.
     4. Test by opening local version of chef.html in browser.
     5. Save, commit, pull request.
10) Eventually we release new software with new languages and translations, i.e. new localisations.
    - lime-packages = 1-2x/year
    - lime-app = together with lime-packages, although can be installed separately
    - chef = instant? weekly?