# Markdown preview

*Update*. For [reStructuredText][rst] there is an excellent [restview][restview]
library which works out-of-the-box.


It took half-a-workday time to find suitable solution to be able to somehow
preview rendered markdown files.

[GitHub][github] uses [GitHub Flavored Markdown][ghmd] so before committing
anything to the [Jukoro][jukoro] reporsitory it would be great to take a quick
look at rendered markdown files.

My editor of choice is [vim][vim] and I have rather strong intension to keep it
as is.

I prefer simple solutions meaning right after save I can see changes rendered
in browser (I don't need really live preview to see changes while typing).
Browser-based editors don't suit me due to the fact you have to edit a file in
browser.

I looked at [Atom][atom] and almost decided to use it but a small annoyance
forced me to look better for other ways (the annoyance is about caret
disappearing in editor after switching from and back to it).
For me there is one more problem with it.
I'd like to turn off bold font in editor (the way I get used to working
in terminal) but I have no luck searching for this feature in preferences.

There are several [nodejs][node]-based solutions but with most of them I had a
problem with live update (for some reason it was not working as expected by me).

[markdown-live][mdlive] is the solution that I chose.
It works great and have a side panel to quickly switch between markdown-files.
Just great.


[github]: https://github.com/
[ghmd]: https://help.github.com/articles/github-flavored-markdown/
[jukoro]: https://github.com/ysegorov/jukoro/
[vim]: http://www.vim.org/
[atom]: https://atom.io/
[node]: https://nodejs.org/
[mdlive]: https://github.com/mobily/markdown-live
[restview]: https://mg.pov.lt/restview/
[rst]: http://docutils.sourceforge.net/rst.html
