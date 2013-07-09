---
layout: default
title: On Git's Shortcomings
---

Git receives a lot of positive press. [There][pro-1] [are][pro-2]
[countless][pro-3] [websites][pro-4], [articles][pro-5], [and][pro-6]
[blog][pro-7] [posts][pro-8] [dedicated][pro-9] [to][pro-10] [the][pro-11]
[adulation][pro-12] [of][pro-13] [Git][pro-14]. It's plenty easy to find a list
of reasons to use Git. It's much harder to find a list of substantial reasons
not to use Git.

  [pro-1]: http://git-scm.com/about
  [pro-2]: https://github.com/
  [pro-3]: http://thkoch2001.github.io/whygitisbetter/
  [pro-4]: http://stackoverflow.com/questions/871/why-is-git-better-than-subversion
  [pro-5]: http://www.gitguys.com/topics/why-use-git-instead-of-a-legacy-version-control-system/
  [pro-6]: http://tumble.jeremyhubert.com/post/2752834826/3-reasons-i-use-git-instead-of-svn
  [pro-7]: http://blog.teamtreehouse.com/why-you-should-switch-from-subversion-to-git
  [pro-8]: http://mendicantbug.com/2008/11/30/10-reasons-to-use-git-for-research/
  [pro-9]: http://jeetworks.org/node/11
  [pro-10]: http://www.git-tower.com/blog/8-reasons-for-switching-to-git/
  [pro-11]: http://blogs.atlassian.com/tag/git/
  [pro-12]: http://www.linuxjournal.com/content/git-revision-control-perfected
  [pro-13]: http://sixrevisions.com/web-development/introductory-guide-to-git-version-control-system/
  [pro-14]: http://coding.smashingmagazine.com/2011/07/26/modern-version-control-with-git-series/

That's not surprising. There's an obvious [selection bias][] at play. Those who
spend more time with Git understand it better and are more likely to extol its
virtues. Conversely, those who are turned off by Git early are unlikely to make
well informed arguments against it.

  [selection bias]: http://en.wikipedia.org/wiki/Selection_bias

Unable to find a complete list of Git's weaknesses, I have attempted to compile
a list myself. Don't get me wrong, I'm a proponent of Git ([this site][] is on
[GitHub][] after all), but we must be truthful about Git's limitations. This
list (with a few exceptions) is about Git alone. I will avoid making
comparisons to other version control systems unless relevant.

  [this site]: https://github.com/peterlundgren/peterlundgren.github.io
  [GitHub]: https://github.com/


Ease of Use
-----------

Let's get this one out of the way first. [Almost][con-1] [all][con-2]
[of][con-3] [the][con-4] [complaints][con-5] [that][con-6] [I][con-7]
[could][con-8] [find][con-9] fell into this category. I don't have much to add
to this subject that hasn't already been said.

  [con-1]: http://www.forouzani.com/disadvantages-of-git.html
  [con-2]: http://journal.dedasys.com/2009/01/06/git-is-a-pain-in-the-ass
  [con-3]: http://kfsone.wordpress.com/2010/05/06/git-a-review-by-a-not-gonna-use-it-er/
  [con-4]: http://mkaz.com/web-dev/dont-be-a-git-use-subversion
  [con-5]: https://steveko.wordpress.com/2012/02/24/10-things-i-hate-about-git/
  [con-6]: http://ventrellathing.wordpress.com/2013/01/25/git-a-nightmare-of-mixed-metaphors/
  [con-7]: http://www.g33klaw.com/2012/04/my-problem-with-git-no-abstraction/
  [con-8]: http://unspecified.wordpress.com/2010/03/26/why-git-aint-better-than-x/
  [con-9]: http://programmers.stackexchange.com/questions/111633/what-does-svn-do-better-than-git

In brief, Git has a complex information model and it doesn't really abstract
that from the user. Git's model is comprised of directed acyclic graphs,
commits, trees, blobs, branches, tags, and remotes. Git has a staging area, a
stash, and a reflog. All in all, Git has 145 commands, but there's no `git
undo`. Sure, you don't need to know all of them, but you do need to know at
least 13 to be minimally productive (`add`, `branch`, `checkout`, `clone`,
`commit`, `diff`, `fetch`, `help`, `init`, `log`, `merge`, `push`, `status`).
Even [specifying Git revisions][gitrevisions(7)] needs its own manpage.

  [gitrevisions(7)]: https://www.kernel.org/pub/software/scm/git/docs/gitrevisions.html

Git is complicated. I welcome the complexity because it brings powerful
features along with it. However, if I wasn't a professional software developer,
I might look elsewhere.


Access Control
--------------

Git doesn't concern itself with access control. It defers that job to the file
system or ssh. Because of this, the ability to restrict access to Git
repositories is severely limited.

*   **Read Access** - You cannot restrict read access to specific files,
    directories, or branches within a single repository with Git. You can
    either clone the entire repository or none of it. You need to arrange
    things such that setting read permissions per-repository is sufficient.

*   **Write Access** - Options for restricting write access aren't quite as
    limited. While Git doesn't support write access control out of the box, it
    does provide [hooks][] that can reject pushes. So, third party tools can
    add write access control to Git. [gitolite][], for example, can
    [restrict write access][] to files, directories, or branches. It can also
    restrict force pushes.

    Unfortunately, this option ties you to a particular third party tool. Want
    to migrate your gitolite access control to GitHub? Too bad.

  [hooks]: http://git-scm.com/book/en/Customizing-Git-Git-Hooks
  [gitolite]: http://gitolite.com/gitolite/
  [restrict write access]: http://gitolite.com/gitolite/admin.html#conf


Obliterate
----------

Some version control tools have a way of completely removing files from the
repository, sometimes called obliterate. Obliterate is more than just delete.
It must purge the data from the repository and its history completely, as if it
was never there in the first place. There are two reasons you might want to do
this:

*   **Confidential Information** - If confidential information is ever
    committed to a repository, deleting it isn't enough. It will remain
    recoverable to anyone with read access to the repository; that's the point
    of version control. Depending upon the level of confidentiality and the
    level of exposure of the repository, obliterate may be necessary.

*   **Large (Measured in Bytes) Mistakes** - If a file is accidentally
    committed to a Git repository and then removed by a later commit, a
    snapshots of that file will forever live in the repository. If that file
    was both added by mistake and very large, this could be a problem.
    Obliterate could remove it for good.

I think Git excels in preventing these problems by having multiple steps to
catch and correct a mistake before it ends up in the central repository. You
need to `add`, `commit`, and `push` (compared to just `commit` with SVN) before
your mistake ends up public.

On the other hand, once it's public, it's public for good. Git makes
cryptographic guarantees that ensure that if someone tries to rewrite history
to obliterate a file, every clone of that repository will notice at the next
fetch (perhaps to the [ire][] of your fellow developers).

  [ire]: http://git-scm.com/book/en/Git-Branching-Rebasing#The-Perils-of-Rebasing

This isn't to suggest that obliterate is trivial in centralized version control
tools. You still have to worry about all the working copies in addition to the
central repository. However, with Git, the problem is more complicated thanks
to locally attached history in every clone.


Locks
-----

Git does not support locking files. How could it? To what central authority
does a distributed version control system go to to obtain a lock? `git lock`
would be an oxymoron.

That said, sometimes locks are needed. Consider binary files that can't be
merged (video, images, CAD drawings, video game assets, etc.). Fundamentally,
each developer needs to take turns working on such files. Locking a file can be
the right solution. For some, locking is a critical feature.

Git doesn't support locking, but [veracity][], another distributed version
control tool, does. If you want to stick with Git, [gitolite][] learned how to
[lock files][] in [2012][]. So, this isn't an intractable problem, but it's
worth watching out for. At the very least, you'll need to configure an
additional tool to support locking.

  [veracity]: http://veracity-scm.com/
  [lock files]: http://gitolite.com/gitolite/locking.html
  [2012]: https://github.com/sitaramc/gitolite/commit/06d3398fb0ef36c835287b73b2794209ecc1eb49


Very Large Repositories
-----------------------

Git stores snapshots of the entire history of a repository locally when you
clone. Disk space is cheap and still getting cheaper. More likely, the
bottleneck is network speed. If you need to clone a large repository over the
network, it's going to take a while. Megabytes, no problem. Gigabytes,
manageable. Terabytes, don't bother (in 2013). This is made more troublesome by
the lack of partial clones.

*   **Partial Clone by Path** - Partial clone by path is not possible in Git.

    [git-archive(1)] lets you download part of a repository by path as a *tar*
    or *zip*. But that's all you get, some files, not a working repository.

    Git learned how to do [sparse checkout][] in version 1.7. This could
    definitely help in some repositories, but it only impacts `checkout`, not
    `clone`. So, it will save some disk space, but no bandwidth. The
    documentation for sparse checkout is burried in [git-read-tree(1)].

*   **Partial Clone by Branch** - `git clone --single-branch` makes it possible
    to clone only a selected branch. However, this will not save much
    bandwidth unless the branches in the remote repository are very far
    diverged. This can be useful in situations like [GitHub Pages][] where you
    have an orphaned branch (`gh-pages`) with different content than the rest
    of your branches.

*   **Shallow Clone** - Git allows shallow clones, which only clone recent
    history, with `git clone --depth <depth>`. However, they come with
    significant limitations. From [git-clone(1)][]:

    > A shallow repository has a number of limitations (you cannot clone or
    > fetch from it, nor push from nor into it), but is adequate if you are
    > only interested in the recent history of a large project with a long
    > history, and would want to send in fixes as patches.

    This can drastically reduce the time to clone a repository with long
    history, particularly for content that doesn't delta-compress well.

  [git-archive(1)]: https://www.kernel.org/pub/software/scm/git/docs/git-archive.html
  [sparse checkout]: http://jasonkarns.com/blog/subdirectory-checkouts-with-git-sparse-checkout/
  [git-read-tree(1)]: https://www.kernel.org/pub/software/scm/git/docs/git-read-tree.html
  [GitHub Pages]: http://pages.github.com/
  [git-clone(1)]: https://www.kernel.org/pub/software/scm/git/docs/git-clone.html

That said, Git's delta-compression can help quite a bit for most projects and
the results may surprise you. After a recent SVN to Git migration, one of our
repositories went from a 5 minute 8 second `svn checkout` to a 4 minute 17
second `git clone`. I did not expect that.

Aside from file size concerns, Git commands can start to [perform poorly][] in
very large repositories (many commits, many files). In short, Git becomes
unusable around 10^6 of each.  That's about 2 orders of magnitude larger than
the [Linux Kernel][] which the Git community tends to think of as a large
repository. The general advice here is to split up your repository before
things get that bad, but that advice can seem awfully hollow to an organization
that operates successfully with such a super-repository.

  [perform poorly]: http://thread.gmane.org/gmane.comp.version-control.git/189776
  [Linux kernel]: https://github.com/torvalds/linux

If you're concerned about performance, some of these [client side][] or [server
side][] benchmarks might interest you. If you're considering migrating a
repository larger than the [Linux kernel][] to Git, you should probably run
your own benchmarks.

  [client side]: https://git.wiki.kernel.org/index.php/GitBenchmarks
  [server side]: http://draketo.de/proj/hg-vs-git-server/test-results.html


Large Number of Contributors to One Branch
------------------------------------------

This point takes a little explaining since it is counter-intuitive. Git works
great for large projects with many contributors. Thanks to its distributed
nature, you don't have to give everyone write access to a central repository.
Because of this, Git is fantastic for open source projects. However, if you
want to give *many* people write access to a single branch on a single
repository, you might run into trouble.

Consider what happens when two people (Alice and Bob) try to push to the same
branch on the same remote at the same time. Let's say Alice gets in first and
successfully updates the branch. Bob's push gets rejected because the remote
branch is no longer reachable from the snapshot he's trying to push. This
ensures that Alice's commits don't get abandoned when Bob pushes. This is a
good thing, but it can become a bottleneck.

Bob now has to fetch, redo his merge, and then try the push again. This process
will take a minute or two. It will take longer in larger repositories and
longer still if you [sign commits][]. This limits the rate of pushes possible
to a single branch. A branch receiving a dozen or more pushes per hour peak is
high traffic, but not unfathomable. 100 pushes per hour is infeasible.

  [sign commits]: http://mikegerwitz.com/papers/git-horror-story.html

As a counter point, Git makes it painless to branch and merge, so it's probably
worth asking why you need to push that often to one branch.


Other
-----

Finally, here are a couple of minor points that deserve mention.

*   **Large Files** - [git-annex][] and [git-bigfiles][] are two efforts to try
    and deal with problems related to large files in Git.

*   **Empty Directoryes** - Git doesn't support empty directories. The standard
    [work-around][] is to add an empty `.gitignore` to the directory and commit
    that instead.

*   **Revision Numbers** - Git doesn't have revision numbers, it has 40
    character hashes. Because of this, a lexicographic sort of revisions no
    longer makes sense. If you are used to appending a [revision number][] to a
    build artifact and having a meaningful sort, you may need to rethink your
    strategy.

*   **SVN Externals Equivalent** - If I had written this post much sooner, this
    would have been an issue. Submodules used to be unable to track branches;
    they had to integrate with a specific commit. This left them less flexible
    than [SVN externals][]. [Git 1.8.2][] recently added this feature. Problem
    solved.

  [git-annex]: http://git-annex.branchable.com/
  [git-bigfiles]: http://caca.zoy.org/wiki/git-bigfiles
  [work-around]: http://stackoverflow.com/questions/115983/how-do-i-add-an-empty-directory-to-a-git-repository
  [revision number]: http://semver.org/
  [SVN externals]: http://svnbook.red-bean.com/en/1.0/ch07s03.html
  [Git 1.8.2]: https://github.com/git/git/blob/master/Documentation/RelNotes/1.8.2.txt


So What?
--------

These shortcomings are a non-issue for many developers. And most of these
weaknesses can be avoided without too much trouble. Still, if you are looking
to use Git on a new project or migrate an existing project to Git, consider
these limitations. Decide for yourself if any of them will be problematic for
your particular use case.
