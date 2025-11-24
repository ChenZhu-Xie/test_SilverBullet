# Hello 👋

Welcome to the wondrous world of SilverBullet. A world that once you discover and appreciate, you’ll never want to leave.

_One of us!_

[[aslkjwer⚓|🧑‍🤝‍🧑4]]${forthRef("aslkjwer")}${backRefs_noSelf("aslkjwer",4)}

You can go ahead and dele
te this placeholder content in a second, but before you do, let me quickly show you around.

We’ll do this through a list of **silver bullets**.

_Ah, so that explains the name!_

* Click on the page picker (book icon) icon at the top right, or hit `Cmd-k` (Mac) or `Ctrl-k` (Linux and Windows) to open the **page picker**.
  * Type the name of a non-existent page to create it.
  * Folders are implicitly created by putting slashes (`/`) in the name (even on Windows), e.g. `My Folder/My Page`. Don’t worry about folders existing, SilverBullet will automatically create them if they don’t.
* Click on the terminal icon (top right), or hit `Cmd-/` (Mac) or `Ctrl-/` (Linux and Windows) to open the **command palette**. From here you can run various useful and perhaps less useful commands.
* Select some text and hit `Cmd-b` (Mac) or `Ctrl-b` (Windows/Linux) to make it **bold**, or `Cmd-i` (Mac) or `Ctrl-i` (Windows/Linux) to make it _italic_.
* You can create links to pages using the `[[other page]]` syntax, for instance: `[[aspiring page]]`. When you link to a non-existent page it will initially show up in orange (to indicate it does not yet exist), but once you click it — you will create the page automatically (only for real when you actually enter some text).
* Start typing `:party` to trigger the emoji picker 🎉
* Type `/` somewhere in the text to invoke a **slash command**.
* If this matches your personality type, there is also an `Editor: Toggle Vim Mode` command to switch to Vim mode. If you cannot figure out how to exit it (classic vim issue), just run that same command again. _Phew!_
* As you may have noticed, while SilverBullet uses [markdown](https://www.markdownguide.org/) underneath, it also “live previews” it to give a bit more of a WYSIWYG feel. You can always move your cursor “into” the underlying code by `Alt-clicking` it (even links and widgets, which we’ll get to later).

[[📌3]]${forthRef("👋")}

While SilverBullet implements (most) of [CommonMark](https://commonmark.org/), it also adds a few extensions that are SilverBullet-specific. Most notably, it adds the `${Lua expression}` syntax to render Lua expressions inline.

Using SilverBullet’s Lua APIs, you can do all kinds of cool stuff. For instance, query your space for the last 3 modified pages. _Exciting!_

${query[[from index.tag "page" order by lastModified desc limit 3]]}

There’s a whole world out there to explore, but let’s not get ahead of ourselves. First, have some fun and add some content to your fresh space.

Then we’ll talk.

# What next?
You can find more information on SilverBullet’s feature set on its [official website](https://silverbullet.md/). Also, be sure to join the [SilverBullet community](https://community.silverbullet.md/) to interact with fellow SilverBullet explorers.
