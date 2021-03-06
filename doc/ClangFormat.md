# MRPT codebase to comply with ClangFormat

The following instructions offer a summary of the formatting/style changes
applied on <TODO Add date>,
as well as handy solutions for solving formatting problems and for making this
transition as smooth as possible.

## Line-wrapping

MRPT code is to be wrapped at **80** lines. This has the following consequences:

- Long inline doxygen comments `//!< ...` are to be wrapped up and indented at
    the same line. This is not most likely not what the developer wants. To get
    around this replace them standard `/** ... */` comments or any other
    alternative outlined in the [doxygen
    docs](https://www.stack.nl/~dimitri/doxygen/manual/docblocks.html).

    For bulk reformatting of these comments one may use `sed` as in [this
    script](https://github.com/bergercookie/clang_git_format/blob/master/scripts/convert_inline_doxygen_comments.sh).

    **WARNING:** The `/**...*/` snippet should be placed on the **previous**
    line from the variable, directive that is for. This is **opposite** to the
    convention for the `//!<...` commenting which is placed to the right of the
    variable/directive.

- Long inline standard C comments `// ...` weren't  been replaced prior to
    running clang-format in the codebase. If these exceeded the chars limit,
    they were wrapped in the next line, which, again, isn't that the original
    developer would have wanted. However, they don't affect either the doxygen
    documentation (they aren't parsed at all) or the code compilation, so they
    are left (wrapped) for now. If you encounter such a case, please
    manually fix the comments (e.g., put them prior to the directive they are
    meant for) and make a PR.


## Updating branches prior to reformatting changes

In case you are working based on a commit prior to the [major reformatting
change](TODO add link to commit) you 'll have to comply with the formatting
changes prior to making your PR to the repository (otherwise the corresponding
travis test fails). The procedure is outlined below:

- Use the reformat_branch functionality of clang_git_format script. We assume
    that the, already reformatted branch you are about to make the PR to, is
    called `mrpt-2.0-devel`. We also assume that you are working on a separate
    branch named `feature`. If that's not the case:

    1. Make a `feature` branch at your latest commit and check that out:

      ```sh
      git checkout -b feature
      ```
    2. Pull the changes from the `upstream/mrpt-2.0-devel` branch:

    ```sh
    git fetch upstream mrpt-2.0-devel
    git checkout mrpt-2.0-devel
    # WARNING: This overwrites whatever separate work you have in mrpt-2.0-devel
    # branch. You should have your work in a separate *feature* branch.
    git reset --hard upstream/mrpt-2.0-devel
    ```

    Checkout your `feature` branch again.

    3. Rebase your `feature` branch right prior to the *major reformat* commit
       (TODO - Add commit ID)

    4. Convert your `//!<` comments to `/**...*/`. Do this by running
       `scripts/convert_dox_comments.sh` on the range of commits of your feature
       branch. Run it from the MRPT root dir. The start commit ID is the one at
       which your branch starts diverging from the mrpt-2.0-devel branch.

       ```sh
       start=$(git merge-base feature mrpt-2.0-devel)
       scripts/convert_dox_comments.sh "${start}" HEAD
       ```

    5. Apply clang-format to your changes. Run `clang_format_codebase` from the
       MRPT root directory, again providing the start and end commits. 

       ```sh
       # script expects to find a clang-format 3.8 executable in the PATH
       scripts/clang_format_codebase.sh ${start}" HEAD
       ```

    6. **Merge** your code in the mrpt-2.0-devel branch. There may be conflicts
       in this step. If these occur in files that you didn't modify
       (indentation/style changes) just discard them:

       ```sh
       git checkout --theirs -- <path to file> ...
       ```

    7. Make sure that the style of the codebase is intact:

       ```sh
       TASK=LINT_ALL travis/check_style.sh
       ```

    TODO - Revisit these instructions - Further testing on this

## Comply new code with ClangFormat

The ClangFormat configuration that MRPT uses can be found [here](TODO - Add
link to .clang-format at ROOT). To check
whether your existing code complies with this, you can run the check_style.sh
script from the MRPT root.

```sh
TASK=LINT_ALL travis/check_style.sh
```

The same script is also used to validate that your changes comply, when you make
a Pull-Request to the MRPT upstream repo.

To format your changes according to the `.clang-format` configuration, either:

- Run `clang-format` directly specifying the location of the modified files
    (beware to use the ClangFormat version specified in
    [clang_git_format/config.py](TODO - Add link to config file):

- Run ClangFormat on the whole codebase. To do that use the
    [clang_format_codebase.sh script](TODO - Add link) from the MRPT root
    directory. Since the MRPT codebase is already formatted, (assuming that
    your work off a commit after the codebase reformat) only your changes are
    going to be modified. Do check the results of ClangFormat before applying
    for a patch


## IDEs

A very important step in the "post-reformat era" is to stick around and
integrate ClangFormat into the daily routine for MRPT development. To that end
there exist plugins, for your editor of choice, that facilitate applying the
ClangFormat configuration while concurrently modifying/adding code.

- [Vim](https://github.com/rhysd/vim-clang-format)

    A sample Vim configuration is given in [this
    snippet](https://gist.github.com/bergercookie/9a2e96e19733b32ca55b8e2940eaba2c)

- [Visual Studio 2017](https://marketplace.visualstudio.com/items?itemName=HansWennborg.ClangFormat)
- [Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=xaver.clang-format)
- [Emacs](https://llvm.org/svn/llvm-project/cfe/trunk/tools/clang-format/clang-format.el)
- [Sublime](https://github.com/rosshemsley/SublimeClangFormat)
- [Atom](https://atom.io/packages/formatter-clangformat)
- [XCode](https://github.com/mapbox/XcodeClangFormat)

The plugin of your choice should:

- Format code based on the `.clang-format` file placed in the MRPT root
    directory.
- Use the ClangFormat **3.8.0** version. This is because part of the
    configuration directives are not compatible with previous ClangFormat
    versions.

Refer to the docs of the corresponding plugin for tweaking its behavior.
Also, do check the [MRPT Coding style](https://github.com/MRPT/mrpt/blob/master/doc/MRPT_Coding_Style.md)
to minimize the content that ClangFormat modifies

TODO - Add to this. Any knowledge of the aforementioned IDE plugins?

## Relevant links

- [Discussion issue](https://github.com/MRPT/mrpt/issues/520)
- [Pull-Request](https://github.com/MRPT/mrpt/pull/556)

## TODOs

- [X] Branch 2.0 to compile successfully after ClangFormat
- [X] Add travis test file
- [X] Enable to lint one's work offline - prior to PR/commit - see the
    travis/check_syntax.sh file
- [X] Add info on IDEs
- [ ] Add instructions for porting code prior to major reformat-day
- [ ] Update the links
- [ ] Solve all TODOs (how meta! :-) )


