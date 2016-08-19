Apache Spark 2 번역 참여 방법

어서 오세요~ 환영합니다. 본 문서는 스파크2 번역에 참여하시는 분들을 위한 소개입니다.
기본적으로 https://spark.apache.org/docs/latest/ 문서에 걸려있는 모든 링크에 대해 번역 하는 것을 목적으로 합니다.
사전 지식으로 git, github 사용법을 이해하셔야 합니다. 마크다운(md) 문서 작성에 대해서도 이해가 필요하지만 실제 md 파일을 열어보시면 직관적으로 바로 아실 수 있기 때문에 따로 공부하지 않으셔도 될 것 같습니다.
git, github 공부 추천 링크 : http://nolboo.kim/blog/2013/10/06/github-for-beginner/
https://github.com/oopchoi/spark-docs 접속해서 Fork
자신의 원격 깃헙에서 로컬로 클론(체크아웃)
docs 경로에 있는 md 파일을 번역하고 커밋, 푸쉬
풀리퀘스트
주의) 여러 사람이 공동으로 작업하기 때문에 같은 파일을 수정하면 충돌이 납니다. 따라서 반드시 이슈를 등록 하고 진행중인 번역 파일에 대해 작성해주셔야 합니다.

이슈 등록 하는 방법 : https://medium.com/@oopchoi/%EC%9D%B4%EC%8A%88-%EB%93%B1%EB%A1%9D-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-7ff328584fb7#.ikbghwtt7
번역을 마치고 풀리퀘스트로 머지가 완료되면 이슈를 close 하시면 됩니다.
문서 빌드 방법 : 현재 API docs 부분을 빌드하면 에러가 발생하므로
$ SKIP_API=1 jekyll build
를 사용해서 빌드하시면 기본 문서들은 생성이 잘 됩니다.
빌드 방법 : https://medium.com/@oopchoi/%EB%A7%A5os%EC%97%90%EC%84%9C-spark-docs-%EB%B9%8C%EB%93%9C%ED%95%98%EA%B8%B0-d8794ea09c3e#.7mgtxkf08




Welcome to the Spark documentation!

This readme will walk you through navigating and building the Spark documentation, which is included
here with the Spark source code. You can also find documentation specific to release versions of
Spark at http://spark.apache.org/documentation.html.

Read on to learn more about viewing documentation in plain text (i.e., markdown) or building the
documentation yourself. Why build it yourself? So that you have the docs that corresponds to
whichever version of Spark you currently have checked out of revision control.

## Prerequisites
The Spark documentation build uses a number of tools to build HTML docs and API docs in Scala,
Python and R.

You need to have [Ruby](https://www.ruby-lang.org/en/documentation/installation/) and
[Python](https://docs.python.org/2/using/unix.html#getting-and-installing-the-latest-version-of-python)
installed. Also install the following libraries:
```sh
    $ sudo gem install jekyll jekyll-redirect-from pygments.rb
    $ sudo pip install Pygments
    # Following is needed only for generating API docs
    $ sudo pip install sphinx
    $ sudo Rscript -e 'install.packages(c("knitr", "devtools", "roxygen2", "testthat"), repos="http://cran.stat.ucla.edu/")'
```
(Note: If you are on a system with both Ruby 1.9 and Ruby 2.0 you may need to replace gem with gem2.0)

## Generating the Documentation HTML

We include the Spark documentation as part of the source (as opposed to using a hosted wiki, such as
the github wiki, as the definitive documentation) to enable the documentation to evolve along with
the source code and be captured by revision control (currently git). This way the code automatically
includes the version of the documentation that is relevant regardless of which version or release
you have checked out or downloaded.

In this directory you will find textfiles formatted using Markdown, with an ".md" suffix. You can
read those text files directly if you want. Start with index.md.

Execute `jekyll build` from the `docs/` directory to compile the site. Compiling the site with
Jekyll will create a directory called `_site` containing index.html as well as the rest of the
compiled files.

    $ cd docs
    $ jekyll build

You can modify the default Jekyll build as follows:
```sh
    # Skip generating API docs (which takes a while)
    $ SKIP_API=1 jekyll build
    
    # Serve content locally on port 4000
    $ jekyll serve --watch
    
    # Build the site with extra features used on the live page
    $ PRODUCTION=1 jekyll build
```

## API Docs (Scaladoc, Sphinx, roxygen2)

You can build just the Spark scaladoc by running `build/sbt unidoc` from the SPARK_PROJECT_ROOT directory.

Similarly, you can build just the PySpark docs by running `make html` from the
SPARK_PROJECT_ROOT/python/docs directory. Documentation is only generated for classes that are listed as
public in `__init__.py`. The SparkR docs can be built by running SPARK_PROJECT_ROOT/R/create-docs.sh.

When you run `jekyll` in the `docs` directory, it will also copy over the scaladoc for the various
Spark subprojects into the `docs` directory (and then also into the `_site` directory). We use a
jekyll plugin to run `build/sbt unidoc` before building the site so if you haven't run it (recently) it
may take some time as it generates all of the scaladoc.  The jekyll plugin also generates the
PySpark docs using [Sphinx](http://sphinx-doc.org/).

NOTE: To skip the step of building and copying over the Scala, Python, R API docs, run `SKIP_API=1
jekyll`.
