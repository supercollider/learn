# Super Collider Tutorials
The repository for the official SuperCollider online tutorial and reference information. Currently
this is in the early stages of development.

Get Involved
-------

If you want to get involved then contact us either on the [SuperCollider Forum][forum] or the
[SuperCollider Slack][slack] channel. Any level of involvement is fine. If you want to write a
tutorial by yourself then we won't stop you - equally if all you can do is proof read, or suggest
resources, that's also helpful.

Tutorial Standards
--------

Documents should have a max line length of 100 columns and be written in markdown.

Code should follow the SuperCollider style guidelines here:
https://github.com/supercollider/supercollider/wiki/Code-style-guidelines

Cookbook entries should address a particular problem that people might run into and show them how
it can be solved. It is perfectly okay to show multiple solutions, but these _must_ be practical
solutions.

## Forking this repository
You will need to create a fork of this repository in order to make edits to your fork, and then submit pull requests containing your changes to this repository.
1. Create a github account [here](https://github.com/join) if necessary
1. Login to github [here](https://github.com/login)
1. Navigate to this page
1. Click the `Fork` link at the top right of the page

Github will create a fork of this repository in your account. You will now be able to clone it and make changes to it like any other git repository.

## Submitting a pull request
Now that you have made some changes to your fork, you will want to submit a pull request back to this repository. To do so:
1. Ensure your changes are committed and pushed to a branch in your fork
1. Navigate to that branch in your fork in github
1. Click the `New pull request` button next to the branch name near the top left
1. Ensure that the `base fork` and `base` drop-downs are set to `supercollider/learn` and `master` respectively
1. Click the `Create pull request button`
1. Enter a title and comment
1. Click the `Create pull request` button

You will be redirected to the `supercollider/learn` homepage on github. Your pull request has been submitted and will be considered for inclusion in the repository.

Requirements
-------

The tutorials are currently built using [mdBook]. On Windows and Linux you just need to download
the binary from their site and update your path so that it points to it. Unfortunately they do not
have a download available for OSX. Fortunately building MDBook is super easy (just follow the
instructions). If for some reason you are unable to build it - then contact me on [Slack][slack]
and I'll send you a binary.

Build
-------

[mdBook] is a clone of the original [gitbook] and uses [markdown]. So long as you're familiar with
[markdown] then you really shouldn't have any issues as the [mdBook guide] is pretty decent.

Currently there are four tutorials in the src file. The tutorials are built separately (this will
change at some future point). The src code for each book is in ```src/[book-name]```. The html will
be generated in a sub-folder of the ```book``` directory in the root folder.

Useful commands:

+ ```mdbook build``` - Build the tutorial.
+ ```mdbook build -o``` - Build the tutorial and open it in your default browser.
+ ```mdbook watch``` - Build the tutorial automatically whenever any of the
  files are changed.
+ ```mdbook serve -o``` - Preview the tutorial at ```http:\\localhost:3000```

License
-------

All content in this repository is licensed under a Creative Commons
Attribution-ShareAlike 4.0 International License.

You should have received a copy of the license along with this work.  If not,
see <http://creativecommons.org/licenses/by-sa/4.0/>.

[mdBook Guide]: https://rust-lang-nursery.github.io/mdBook/
[mdBook]: https://github.com/rust-lang-nursery/mdBook
[forum]: https://scsynth.org/
[slack]: https://join.slack.com/t/scsynth/shared_invite/enQtMzk3OTY3MzE0MTAyLWY1ZGE1MTJjYmI5NTRkZjFmNjZmNmYxOWI0NDZkNjdkMzdkNjgxNTJhZGVlOTEwYjdjMDY5OWM0ZTA4NWFiOGY
[gitbook]: https://toolchain.gitbook.com/
[markdown]: https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet
