# `gaussEx.sty` - An expl3 extension of `gauss.sty`
The LaTeX3 package `gaussEx.sty` is ...
- ... a reimplementation of the `gauss.sty` package [on CTAN](https://ctan.org/pkg/gauss) in the LaTeX3 programming layer (expl3)
- ... extending the base functionality of `gauss.sty` by adding support for vertical and horizontal dividers, custom row and column separation and a few more options for typesetting the matricies used for visualizing operations of the Gaussian elimination

What someone who has already used `gauss.sty` and now wants to make use of the new feature set of `gaussEx.sty` should pay attention to, is listed and explained in the section [Migrating from `gauss.sty`](#migrating-from-gausssty). The section [New features](#new-features) fully explains the new features of `gaussEx.sty` with a lot of concrete examples. 

# Roadmap
- [x] Reimplement `gauss.sty` in expl3 
- [x] Implement the old `gauss.sty` macros and dimensions as options
- [ ] Implement the option mechanism for the individual `gexmatrix` environments 
- [ ] Implement matrix classes 
- [ ] Implement the new options 

# Migrating from `gauss.sty` 
## What is now deprecated
`gaussEx.sty` was written with backwards compability to `gauss.sty` in mind. Thus, e.g. the `0`-based indexing of rows and columns was kept (In the future we may offer an option to choose either `0`- or `1`-based indexing). Also the implementation of `\mult` was not changed on the document level - although it differs from the implementation of `\add` and `\swapp` and thus introduces an ugly inconsistency. 
But as an effort to follow modern LaTeX3 paradigms, all labels and public lengths have been privatized in the sense that they now are private module keys (and can only be accessed as such). Hence, the following macros cannot be used with `gaussEx.sty` (they are simply not defined): 
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

Furthermore, the following TeX dimensions are also privatized as module keys and can only be accessed as such (they are defined as private expl3 dim variables): 
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
\def\rowaddfromlabel#1{\scriptstyle #1}
```
you now have to use `\SetGexDefaults` by writing
```latex
\SetGexDefaults{
    colarrowsep = .5em,
    rowaddfromlabel = {\scriptstyle #1}
}
```
## Option scope
`\SetGexDefaults` sets options locally (i.e. only in the current TeX group) as "defaults" for the `gexmatrix` environment. Of course, these defaults can be overriden for every single `gexmatrix` environment. For example, one could write
```latex
\begin{gexmatrix}[
    colarrowsep = 1em,
    rowaddfromlabel = {\scriptscriptstyle #1}
]
...
\end{gexmatrix}
```
to "override" the options set by the previous `\SetGexDefaults`. 

## Conflicts with AMSLaTeX
Due to conflicts with earlier versions of AMSLaTeX, `gauss.sty` globally(!) redefined AMSLaTeX environments like `pmatrix` and `Bmatrix`. This is a very ugly solution. Hence, `gaussEx.sty` does never globally (re-)define matrix environments and the `\newmatrix` mechanism from `gauss.sty` is now deprecated and not supported anymore. Delimiters are now explicitely given by the options `ldelim` and `rdelim`, and authors can define _matrix classes_ (see section [Matrix classes](#matrix-classes)). 

 Internally, `gaussEx.sty` uses exactly the same typesetting mechanism that also `gauss.sty` uses with `\newmatrix` but now privately. 

 ## Migrating a project using `gauss.sty` 
 If you already have some large (or small) document that uses `gauss.sty`, you have to go through the following steps to migrate to the `gaussEx.sty` implementation and use all the new features: 
 
1. Replace 
    ```latex
    \begin{gmatrix}[<delim type>]
    ... 
    \end{gmatrix}
    ``` 
    with 
    ```latex
    \begin{gexmatrix}[cls=<delim type>]
    ... 
    \end{gexmatrix}
    ```
    Or, if you just use the same `<delim type>` in your whole document, say 
    ```latex
    \SetGexDefaults{
        cls = <delim type>
    }
    ```
    in the preamble and then just write 
    ```latex
    \begin{gexmatrix}
    ...
    \end{gexmatrix}
    ```
    no need for any `cls` option. 

2. In case you defined your own delimiters with `\gauss.sty` using `\newmatrix`, you have to replace your definition 
    ```latex
    \newmatrix{<left-delim>}{<right-delim>}{<idenfitier>}
    ```
    with 
    ```latex
    \NewGexMatrix{<identifier>}{
        ldelim = <left-delim>,
        rdelim = <right-delim>
    }
    ```

3. If you used some custom implementations for the macros `gauss.sty` provides, e.g. 
    ```latex
    \def\rowmultlabel#1{|\cdot#1}
    \def\rowaddfromlabel#1{\scriptscriptstyle +}
    ```
    you have to set them using `\SetGexDefaults`: 
    ```latex
    \SetGexDefaults{
        rowmultlabel = {|\cdot#1},
        rowaddfromlabel = {\scriptscriptstyle +}
    }
    ```

4. If you set some new values for the TeX dimension, `gauss.sty` provides, e.g. 
    ```latex
    \colarrowsep=1em
    \rowarrowsep=1.5em
    ```
    you have to set them using `\SetGexDefaults`: 
    ```latex
    \SetGexDefaults{
        colarrowsep = 1em,
        rowarrowsep = 1.5em
    }
    ```

# New features

## New matrix options
`gaussEx.sty` does not only replicate the features of `gauss.sty` in expl3, but also _extend_ the features of the original package by adding the following new global styling options: 
- `colsep`: Column separation
- `rowsep`: Row separation 
- `ldelim`: Specifies the left delimiter of the matrix
- `rdelim`: Specifies the right delimiter of the matrix
- `colalign`: `l` (left), `c` (center), `r` (right) for the alignment of the content in a cell. The default ist `c`. There is no analogous key for rows (i.e. `rowalign` is not a key) as there simply is no row alignment in LaTeX. (Maybe cell padding?)
- `visualcenter`: Takes no value. If it is passed to `\SetGexDefaults` or `gexmatrix`, the horizontal dividers will be aligned to the visual center between two rows. This will lead to problems with cells that contain material with a large depth, so this option generally is only recommended when using small cell entries like numbers. 
- `matrixpadding`: Padding of the matrix to the top, bottom, left and right in the format `<top and right and bottom and left>`, `<top and bottom> <left and right>` or `<top> <right> <bottom> <left>` (similar to how CSS is reading padding arguments). 
- `vdivolen`: Overlength of the vertical dividers. If the overlength is larger than the top or bottom padding account for, the padding is automatically increased. 
- `hdivolen`: Overlength of the horizontal dividers. If the overlength is larger than the left or right padding account for, the padding is automatically increased. 
- `cls`: Identifier for a matrix class. `gaussEx.sty` predefines the matrix classes `p` (for `(`, `)` parenthesis), `b` (for `[`, `]` brackets), `B` (for `\lbrace`, `\rbrace` braces), `v` (for `\lvert`, `\rvert` delimiters) and `V` (for `\lVert`, `\rVert` delimiters). 

Additionally, there are two options which can only be applied to `gexmatrix` environments directly and not via `\SetGexDefaults`: 
- `cols`: Styling options inspired by way the `array` environment specifies alignment and vertical dividers. 
- `rows`: Styling options inspired by way the `array` environment specifies alignment and vertical dividers. Maybe in the future, `gaussEx.sty` will also support inline commands such at `\hline` and `\\[<distance>]` to regulate the spacing between rows and horizontal dividers. 

These two options are a little more complicated than the rest but should still be quite straight forward. For example, consider the following matrix
```latex
\begin{gexmatrix}[cls=p]
1 & 2 & 3 \\
4 & 5 & 6 
\end{gexmatrix}
```
Per default the content of each cell is aligned to the center of a cell. To align the first two colums to the right side of each cell and draw a vertical divider between the second and third column, you write
```latex
\begin{gexmatrix}[cls=p, cols=cc|r]
1 & 2 & 3 \\
4 & 5 & 6 
\end{gexmatrix}
```
If you want more spacing between two columns, you should use the `S{<distance>}` specifier. To add more spacing arround the vertical divider (i.e. between the second and third column), you could write 
```latex
\begin{gexmatrix}[cls=p, cols=ccS{10pt}|r]
1 & 2 & 3 \\
4 & 5 & 6 
\end{gexmatrix}
```
or even 
```latex
\begin{gexmatrix}[cls=p, cols=cc|S{10pt}r]
1 & 2 & 3 \\
4 & 5 & 6 
\end{gexmatrix}
```
The row styling works analogously with the exception that there are no alignment options as `l`, `c` and `r`. Instead, one would just write `x`. So to add a horizontal divider between the first and second row and also additional spacing of `10pt`, one would write 
```latex
\begin{gexmatrix}[cls=p, cols=ccS{10pt}|r, rows=xS{10pt}|x]
1 & 2 & 3 \\
4 & 5 & 6 
\end{gexmatrix}
```

## Matrix classes 
Instead of using `\newmatrix`, authors can now define new matrix classes using `\NewGexMatrix`. This command takes two arguments: 
1. The name of the matrix class 
2. The options that should be set under that class 

For example, to create a matrix with `(` and `)` delimiters, one could write
```latex
\NewGexMatrix{p}{
    ldelim = (, 
    rdelim = ) 
}
```
and then use it like 
```latex 
\begin{gexmatrix}[cls=p]
...
\end{gexmatrix}
```
If you now want more padding around the matrix (i.e. more spacing between the matrix and the delimiters), you could simply write 
```latex
\NewGexMatrix{p}{
    ldelim = (, 
    rdelim = ),
    matrixpadding = 2pt 2pt, 
}
```
instead of the previous definition.

## Overriding matrix classes
Already defined matrix classes can easily be overriden. For example, in your preamble, you could write something like 
```latex
\NewGexMatrix{T}{
    ldelim = (, 
    rdelim = ),
    matrixpadding = 2pt 2pt, 
}

\NewGexMatrix{T}{
    ldelim = [, 
    rdelim = ),
    matrixpadding = 1pt 10pt, 
}
```
Of course, also the matrix classes initially defined by `gaussEx.sty` can be redefined. 


## Order of options
The `gexmatrix` environment reads in options in the ordering they were given. For example, 
```latex
\begin{gexmatrix}[matrixpadding = 2pt 2pt, matrixpadding = 1pt]
...
\end{gexmatrix}
```
would first apply the `matrixpadding` option with the value `2pt 2pt` and then override this option with `1pt`. Hence, the last instance of an option is always the one that's being applied. There is no cascading of options happening. 

Similarly, 
```latex 
% Preamble code 
\NewGexMatrix{p}{
    ldelim = (, 
    rdelim = ),
    matrixpadding = 2pt 2pt, 
}

% Document code 
\begin{gexmatrix}[cls=p, matrixpadding = 1pt]
...
\end{gexmatrix}
```
would first compute the matrix class `p`, i.e. set the options `ldelim`, `rdelim` and `matrixpadding` as specified in the  `\NewGexMatrix` definition and then set the `matrixpadding` option to `1pt`. 
