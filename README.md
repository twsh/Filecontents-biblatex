Sharing LaTeX files can be awkward. They can include other files using `\input` and `\include` commands, and they can depend on external files such as bibliographies and packages. The [latexpand](http://www.ctan.org/pkg/latexpand) script will 'flatten' files to make them easier to share. The standard behaviour is to put the contents of `\input` and `\include` in the file and remove any comments. The `--output` option allows you to keep the original file as it was. There are several options, including `--expand-usepackage` which will copy in any loaded packages. This will probably give you a big file but it will be maximally portable.

What latexpand won't do is flatten a biblatex bibliography. One step towards doing that is to make a `.bib` file containing just the things cited in the file. This can be done with [biber](http://sourceforge.net/projects/biblatex-biber/). The command `biber --output-safechars --output_format=bibtex --output_resolve foo.bcf` (where 'foo.bcf' is a previously existing aux file) will generate a `.bib` file containing only the things cited. The `--output_resolve` option tells biber to expand cross references. (I found the biber manual opaque there; this [answer](http://tex.stackexchange.com/a/164328/451) helped me.)

The final step would be to put that bibliography in a `filecontents` environment. Here's a simple example:

```
\begin{filecontents}{\jobname.bib}
@article{Doe2014,
    Author = {Doe, Jane},
    Journaltitle = {The Journal},
    Pages = {1--20},
    Title = {A Title},
    Volume = {1},
    Year = {2014}}
}
\end{filecontents}
\addbibresource{\jobname.bib}
```

When the LaTeX file is run it will produce a `.bib` file containing that information and use it as its source for biblatex. This script will speed up this process by making the `.bib` file by running biber with the right options, and then producing a `.tex` file with the filecontents environment instead of an ordinary `\addbibresource`. The `.bib` file will be deleted afterwards. The script takes a `.tex` file as an argument and has two optional arguments: `--output` is the name of the output file (defaults to 'standalone.tex'); `--bcf` is the path to the `.bcf` file (defaults to the name of the `.tex` file). This results in a single file, after running [latexpand](http://www.ctan.org/pkg/latexpand), if necessary, which can be shared and run on any machine with a standard LaTeX installation.

The script will strip out some of the fields that BibDesk adds but which one perhaps wouldn't want in a file to be shared: 'read', 'date-added', 'date-modified', and 'bdsk-file-1'. I also ran into an odd problem which is that 'url' fields were being replaced with 'link' fields. I don't know why, but I added something that changes them back. The script relies on both [biber](http://biblatex-biber.sourceforge.net) and the [BibTexParser](https://pypi.python.org/pypi/bibtexparser) Python package.
