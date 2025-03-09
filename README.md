# `gaussEx.sty` - An expl3 extension of `gauss.sty`
The LaTeX3 package `gaussEx.sty` is ...
- ... a rewrite of the `gauss.sty` package [on CTAN](https://ctan.org/pkg/gauss) in the expl3 programming layer
- ... extending the base functionality of `gauss.sty` by adding support for vertical and horizontal dividers, custom row and column separation and a few more options for typesetting the matricies used for visualizing operations of the Gaussian elimination

What someone who has already used `gauss.sty` and now wants to make use of the new feature set of `gaussEx.sty` should pay attention to is listed and explained in the first section _Migrating from `gauss.sty`_. The second section _New features_  fully explains the new features of `gaussEx.sty` with a lot of concrete examples. 

# Migrating from `gauss.sty` 
## What is now deprecated
`gaussEx.sty` was written with backwards compability to `gauss.sty` in mind. Thus, e.g. the `0`-based indexing of rows and columns was kept (In the future we may offer an option to choose either `0`- or `1`-based indexing). Also the implementation of `\mult` was not changed on the document level - although it differs from the implementation of `\add` and `\swapp` and thus introduces an ungly inconsistency. But as an effort to follow modern LaTeX3 paradigms, all labels and public lengths have been privatized in the sense that they now are private module keys (and can only be accessed as such). Hence, the following macros cannot be used with `gaussEx.sty` (they are not defined): 
- `\rowmultlabel` 
- `\rowswapfromlabel`
- `\rowswaptolabel`
- `\rowaddfromlabel`
- `\rowaddtolabel`
- `\colmultlabel` 
- `\colswapfromlabel`
- `\colswaptolabel`
- `\coladdfromlabel`
- `\coladdtolabel`

Furthermore, the following TeX dimensions are also privatized as module keys and cannot be accessed anymore: 
- `\colarrowsep`
- `\rowarrowsep`
- `\opskip`
- `\labelskip`
- `\colopminsize`
- `\rowopminsize`

The defaults for these options has been kept from `gauss.sty`. 

## How to set options
When in `gauss.sty` you would write something like 
```latex
\colarrowsep=.5em
\rowaddfromlabel#1{\scriptstyle #1}
```
you now have to use `\SetGexOptions` by writing
```latex
\SetGexOptions{
    colarrowsep = .5em
    rowaddfromlabel = {\scriptstyle #1}
}
```
## Option scope
`\SetGexOptions` sets options locally (i.e. only in the current group) as "defaults" for the `gexmatrix` environment. Of course, these defaults can be overriden for every single `gexmatrix` environment. For example, one could write
```latex
\begin{gexmatrix}[
    colarrowsep = 1em,
    rowaddfromlabel = {\scriptscriptstyle #1}
]
...
\end{gexmatrix}
```

## Conflicts with AMSLaTeX
Due to conflicts with earlier versions of AMSLaTeX, `gauss.sty` globally(!) redefined AMSLaTeX environments like `pmatrix` and `Bmatrix`. This is a very ugly solution. Hence, `gaussEx.sty` does never globally (re-)define matrix environments and the `\newmatrix` mechanism from `gauss.sty` is now deprecated and not supported anymore. Delimiters are now explicitely given by the options `ldelim` and `rdelim`. Internally, `gaussEx.sty` uses exactly the same rendering mechanism that also `gauss.sty` uses with `\newmatrix` but now privately. 

# New features
`gaussEx.sty` does not only replicate the features of `gauss.sty` in expl3, it does also _extend_ the features of the original package by adding the following new global styling options: 
- `colsep`: Column separation
- `rowsep`: Row separation 
- `ldelim`: Specifies the left delimiter of the matrix
- `rdelim`: Specifies the right delimiter of the matrix
- `colalign`: `l` (left), `c` (center), `r` (right) for the alignment of the content in a cell. The default ist `c`. There is analogous key for rows (i.e. `rowalign` is not a key) as there simply is no row alignment in LaTeX. (Maybe Padding?)
- `visualcenter`: Takes no value. If it is passed to `\SetGexOptions` or `gexmatrix`, the horizontal dividers will be aligned to the visual center between two rows. This will lead to problems with cells that contain material with a large depth, so this option generally is only recommended when using small cell entries like numbers. 
- `matrixpadding`: Padding of the matrix to the top, bottom, left and right in the format `<top and bottom>, <left and right>` or `<top>, <right>, <bottom>, <left>`. 
- `vdivolen`: Overlength of the vertical dividers. If the overlength is larger than the top or bottom padding account for, the padding is automatically increased. 
- `hdivolen`: Overlength of the horizontal dividers. If the overlength is larger than the left or right padding account for, the padding is automatically increased. 

Additionally, there are two options which can only be applied to `gexmatrix` environments directly and not via `\SetGexOptions`: 
- `cols`: Styling options inspired by way the `array` environment specifies alignment and vertical dividers. 
- `rows`: Styling options inspired by way the `array` environment specifies alignment and vertical dividers. Maybe in the future, `gaussEx.sty` will also support inline commands such at `\hline` and `\\[<distance>]` to regulate the spacing between rows and horizontal dividers. 

These two options are a little more complicated than the rest but should still be quite straight forward. For example, consider the following matrix
```latex
\begin{gexmatrix}[type=p]
1 & 2 & 3 \\
4 & 5 & 6 
\end{gexmatrix}
```
Per default the content of each cell is aligned to the center of a cell. To align the first two colums to the right side of each cell and draw a vertical divider between the second and third column, you write
```latex
\begin{gexmatrix}[type=p, cols=cc|r]
1 & 2 & 3 \\
4 & 5 & 6 
\end{gexmatrix}
```
If you want more spacing between two columns, you should use the `S{<distance>}` specifier. To add more spacing arround the vertical divider (i.e. between the second and third column), you could write 
```latex
\begin{gexmatrix}[type=p, cols=ccS{10pt}|r]
1 & 2 & 3 \\
4 & 5 & 6 
\end{gexmatrix}
```
or even 
```latex
\begin{gexmatrix}[type=p, cols=cc|S{10pt}r]
1 & 2 & 3 \\
4 & 5 & 6 
\end{gexmatrix}
```
The row styling works analogously with the exception that there are no alignment options as `l`, `c` and `r`. Instead, one would just write `x`. So to add a horizontal divider between the first and second row and also additional spacing of `10pt`, one would write 
```latex
\begin{gexmatrix}[type=p, cols=ccS{10pt}|r, rows=xS{10pt}|x]
1 & 2 & 3 \\
4 & 5 & 6 
\end{gexmatrix}
```