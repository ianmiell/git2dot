# git2dot
Visualize a git repository using the graphviz dot tool.

It is useful for understanding how git works in detail.  You can use
it to analyze repositories before and after operations like merge and
rebase to really get a feeling for what happens. It can also be used
for looking at subsets of history on live repositories.

Here is an example that shows the PNG file generated by test04 in the
test directory.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://cloud.githubusercontent.com/assets/2991242/22413672/a3357722-e66e-11e6-8cc8-332b5123a561.png" alt="test04 example">

Here is a quick rundown of what you are seeing:
 
1. The bisque (tan) nodes are commits. Each commit has the short id, the commit date, the subject (truncated to 32 characters) and the change-id (if it exists). The fields are the same for merged and squashed nodes as well.
2. The light red nodes are merged nodes. They are commits that resulted from a merge (they have 2 or more children).
3. The dark red nodes are squashed nodes. They are the end-points of squashed commit chains. Squashed commit chains do not have any branches, tags, change-ids or merges. They are just a long chain of work. If you turn off squashing, the graph gets really unwieldy. In one case (shown a bit later) there were 155 intermediate nodes!
4. The bluish boxes underneath commit and merge nodes are the branches associated with the commit. There is one box for each branch.
5. The light purple boxes above the commit and merge nodes are the tags associated with the commit. There is one box for each tag.
6. The arrows on the edges show the back (parent) pointer relationships from the repo. Git does not have child references.

It works by running over the .git repository in the current directory
and generating a commit relationship DAG that has both parent and
child relationships.

The generated graph shows commits, tags and branches as nodes.
Commits are further broken down into simple commits and merged commits
where merged commits are commits with 2 or more children. There is an
additional option that allows you to squash long chains of simple
commits with no branch or tag data.

It has a number of different options for customizing the nodes,
using your own custom git command to generate the data, keeping
the generated data for re-use, customizing dot directly and
generating graphical output like PNG, SVG or even HTML files.

Here is an example run:
```bash
   $ cd SANDBOX
   $ git2dot.py --png git.dot
   $ open -a Preview git.dot.png  # on Mac OS X
   $ display git.dot.png          # linux
```
If you want to create a simple HTML page that allows panning and
zooming of the generated SVG then use the --html option like
this.
```bash
   $ cd SANDBOX
   $ git2dot.py --svg --html ~/web/index.html ~/web/git.dot
   $ $ ls ~/web
   git.dot          git.dot.svg      git.html         svg-pan-zoom.js
   $ cd ~/web
   $ python -m SimpleHTTPServer 8090  # start server
   $ # Browse to http://localhost:8090/git.dot.svg
   ```

It assumes that existence of svg-pan-zoom.js from the
https://github.com/ariutta/svg-pan-zoom package.

The output is pretty customizable. For example, to add the subject and
commit date to the commit node names use `-l '%s|%cr'`. The items come
from the git format placeholders or variables that you define using
`-D`. The | separator is used to define the end of a line. The maximum
width of each line can be specified by `-w`. Variables are defined by `-D`
and come from text in the commit message. See `-D` for more details.

You can customize the attributes of the different types of nodes and
edges in the graph using the -?node and -?edge attributes. The table
below briefly describes the different node types:

| Node Type | Brief Description |
| --------- | ----------------- |
|  bedge    | Edge connecting to a bnode. |
|  bnode    | Branch node associated with a commit. |
|  cnode    | Commit node (simple commit node). |
|  mnode    | Merge node. A commit node with multiple children. |
|  snode    | Squashed node. End point of a sequence of squashed nodes. |
|  tedge    | Edge connecting to a tnode. |
|  tnode    | Tag node associated with a commit. |

If you have long chains of single commits use the `--squash` option to
squash out the middle ones. That is generally helpful for filtering
out extraneous commit details for moderately sized repos.

If you find that dot is placing your bnode and tnode nodes in odd
places, use the `--crunch` option to collapse the bnode nodes into
a single node and the tnodes into a single node for each commit.

If you want to limit the analysis to commits between certain dates,
use the `--since` and `--until` options.

If you want to limit the analysis to commits in a certain range use
the `--range` option.

If you want to limit the analysis to a small set of branches or tags
you can use the `--choose-branch` and `--choose-tag` options. These options
prune the graph so that only parents of commits with the choose branch
or tag ids are included in the graph. This gives you more detail
controlled that the git options allowed in the --range command. It
is very useful for determining where branches occurred.

You can choose to keep the git output to re-use multiple times with
different display options or to share by specifying the `-k` (`--keep`)
option.

Use the `-h` option to get detailed information about the available options.

## Example
The following example shows how to use git2dot by creating a git repository
from scratch and then running the tool to create images.

First we create a repository with a bunch of commits and branches.
```bash
git init

echo 'A' >README
git add README
git commit -m 'master - first'
sleep 1

echo 'B' >>README
git add README
git commit -m 'master - second' -m 'Change-Id: I001'
sleep 1

# tag the basis for all of the branches
git tag -a 'v1.0' -m 'Initial version.'
git tag -a 'v1.0a' -m 'Another version.'

git checkout -b branchX1
git checkout master
git checkout -b branchX2

git checkout master
git checkout -b branchA
echo 'C' >> README
git add README
git commit -m 'branchA - first'
sleep 1

echo 'B' >> README
git add README
git commit -m 'branchA - second' -m 'Change-Id: I001'
sleep 1

git checkout master
git checkout -b branchB
echo 'E' >> README
git add README
git commit -m 'branchB - first'
sleep 1

echo 'F' >> README
git add README
git commit -m 'branchB - second'
sleep 1

echo 'B' >> README
git add README
git commit -m 'branchB - third' -m 'Change-Id: I001'
sleep 1

echo 'H' >> README
git add README
git commit -m 'branchB - fourth' -m 'Change-Id: I002'
sleep 1

echo 'I' >> README
git add README
git commit -m 'branchB - fifth'
sleep 1

echo 'J' >> README
git add README
git commit -m 'branchB - sixth'
sleep 1

echo 'K' >> README
git add README
git commit -m 'branchB - seventh'
sleep 1

git checkout master
echo 'L' >> README
git add README
git commit -m 'master - third'
```

You can verify the repo structure using something like `git log`.
```bash
$ git log --graph --oneline --decorate --all
* da0165b (HEAD -> master) master - third
| * 8e3cf50 (branchB) branchB - seventh
| * e0420c1 branchB - sixth
| * f51497b branchB - fifth
| * cee652e branchB - fourth
| * 2fc95e6 branchB - third
| * 9c654d8 branchB - second
| * 33fbc07 branchB - first
|/  
| * 20ea3d2 (branchA) branchA - second
| * 71a0d0c branchA - first
|/  
* ecdc7dc (tag: v1.0a, tag: v1.0, branchX2, branchX1) master - second
* c8ae444 master - first
```

Now run the git2dot tool to generate PNG, HTML and SVG files.
```bash
$ git2dot.py --png --svg --html example.html example.dot
$ ls -1 example.*
example.dot
example.dot.png
example.dot.svg
example.html
```

You can now view the PNG and SVG files using local tools.
```bash
$ open -a Preview example.dot.png  # on Mac
$ display example.dot.png          # on Linux
```

To view the generated SVG file with pan and zoom you must download
the svg-pan-zoom.min.js file from https://github.com/ariutta/svg-pan-zoom
and copy into the current directory.

```bash
$ cp ~/work/svg-pan-zoom-3.4.1/dist/svg-pan-zoom.min.js .
$ ls -1 example* svg*
example.dot
example.dot.png
example.dot.svg
example.html
svg-pan-zoom.min.js
```

Now you need to start a server.

```bash
$  python -m SimpleHTTPServer 8090
```

After that you can browse to http://localhost:8090/example.html and you will see this.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://cloud.githubusercontent.com/assets/2991242/22431235/b585cf7e-e6c5-11e6-8f17-6b99847bfe51.png" width="1100" alt="example">

As you can see, there is a long chain of commits, to run it again using the `--squash` option.

```bash
$ git2dot.py --squash --png --svg --html example1.html example1.dot
```

And browse to http://localhost:8090/example1.html and you will see this.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://cloud.githubusercontent.com/assets/2991242/22431252/c5077344-e6c5-11e6-95b0-54cd02d11aa2.png" width="1100" alt="example1">

Which is a cleaner view of the overall structure.

You will also note that there are two branches and two tags on *ecdc7dc*. They can be collapsed using the `--crunch` option like this.

```bash
$ git2dot.py --crunch --squash --png --svg --html example1.html example1.dot
```
When you browse to http://localhost:8090/example2.html and you will see this.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://cloud.githubusercontent.com/assets/2991242/22431258/c89d7e7c-e6c5-11e6-826e-cf7450b9f125.png" width="1100" alt="example2">

For such a small graph the crunch operation doesn't really make things simpler but for larger graphs where dot may move the branch and tag information around, it can be a much cleaner view.

## Example 2 - pruning the graph

There are two more options you will want to think about for making large graphs
more readable: `--choose-branch` and `--choose-tag`. As described earlier,
they prune the graph so that it only considers the parent chains of the
specified branches or tags. This can be very useful to determining where
branches occurred.

This example shows how it works.

First you create a repository like this.

```bash
git init

echo 'A' >example2.txt
git add example2.txt
git commit -m 'master - first'
sleep 1

echo 'B' >>example2.txt
git add example2.txt
git commit -m 'master - second'
sleep 1

# tag the basis for all of the branches
git tag -a 'v1.0' -m 'Initial version.'
git tag -a 'v1.0a' -m 'Another version.'

git checkout -b branchX1
git checkout master
git checkout -b branchX2

git checkout master
git checkout -b branchA
echo 'C' >> example2.txt
git add example2.txt
git commit -m 'branchA - first'
sleep 1

echo 'D' >> example2.txt
git add example2.txt
git commit -m 'branchA - second'
sleep 1

echo 'E' >> example2.txt
git add example2.txt
git commit -m 'branchA - third'
sleep 1

echo 'F' >> example2.txt
git add example2.txt
git commit -m 'branchA - fourth'
sleep 1

git checkout master
git checkout -b branchB
echo 'G' >> example2.txt
git add example2.txt
git commit -m 'branchB - first'
sleep 1

echo 'H' >> example2.txt
git add example2.txt
git commit -m 'branchB - second'
sleep 1

echo 'I' >> example2.txt
git add example2.txt
git commit -m 'branchB - third'
sleep 1

echo 'J' >> example2.txt
git add example2.txt
git commit -m 'branchB - fourth'
sleep 1
git tag -a 'v2.0a' -m 'Initial version.'

echo 'K' >> example2.txt
git add example2.txt
git commit -m 'branchB - fifth'
sleep 1

echo 'L' >> example2.txt
git add example2.txt
git commit -m 'branchB - sixth'
sleep 1

echo 'M' >> example2.txt
git add example2.txt
git commit -m 'branchB - seventh'
sleep 1

git checkout master
echo 'N' >> example2.txt
git add example2.txt
git commit -m 'master - third'
sleep 1

echo 'O' >> example2.txt
git add example2.txt
git commit -m 'master - fourth'
```

You can confirm its layout like this.

```bash
$ $ git log --graph --oneline --decorate --all --topo-order
* e4bb699 (HEAD -> master, origin/master, origin/HEAD) Add --topo-order to the default range
* 01bb6de Update comments
* c0bf31e Update comments
* f6f32ac (tag: v0.4) Update to describe --choose-* functionality
* 81fc41c v0.4 - added --choose-* support
*   c50cded (tag: v0.3) Merge branch 'master' of https://github.com/jlinoff/git2dot
|\  
| * 3c52eae Update README.md
* | be89609 Add example
|/  
* 680b2e5 Update documentation
* 0b7fed3 Update README.txt
* 47f1430 Initial load
* b4c73c8 Update README.md
* 54632ac Change bedge/tedge color defaults
* 0136d78 Add support for --crunch
* 10eaf83 Update README.md
* 736b75a Fix bug in --align-by-date handling
* 4fac1b8 Fix bug in --align-by-date handling
* fd20bac Fix bug in --align-by-date handling
* e15199f Update README.md
* a4a6fa8 Initial load
* da4e1d3 Update README.md
* ba50fcf Update README.md
* 2a8038c Update README.md
* 800700f Update README.md
* a3a4ae0 Initial commit
```

Create the graph without pruning.

```bash
$ ../git2dot.py --graph-label 'graph[label="example2 - compressed initial state"]' --crunch --squash --png --svg --html example2-2.html example2-2.dot
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://cloud.githubusercontent.com/assets/2991242/22488086/0d34a592-e7c5-11e6-91d8-720f21e357f6.png" width="1100" alt="example2-2">

Create the graph with pruning.

```bash
$ ../git2dot.py --graph-label 'graph[label="example2 - compressed pruned state"]' --choose-branch 'branchA' --choose-tag 'tag: v2.0a' --crunch --squash --png --svg --html example2-4.html example2-4.dot
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://cloud.githubusercontent.com/assets/2991242/22488091/11ae8912-e7c5-11e6-9818-1c8e9c607182.png" width="1100" alt="example2-4">

As you can see, branchB has been completely removed in the second one.

## Hints

1. For large graphs consider using the `--squash` option.
2. For graphs that have multiple branches and tags on the same commits consuder using the `--crunch` option.
3. If you only want to see the combined history of a few branches or tags (like release branches) consider using the `--choose-branch` and `--choose-tag` options to prune the graph.
4. Use the `--since` option if you don't care about ancient history.
5. The `--graph-label` option can be useful and can be very simple: `--graph-label 'graph[label="MY LABEL"]'`.
