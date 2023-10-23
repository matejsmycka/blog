# Guide: How to start with Secret Detection 

## Intro 

When it comes to secret detection, remember, it's not a secret if everyone knows it… especially the hackers. 
Accidentally pushing a password or connection string into remote source control happens even to more seasoned developers. There are many tools that try to solve this problem, this guide describes how to use some of them quickly and easily.

**Secret Detection** utilities can be automated with CI/CD pipelines, GitHub actions and other automation tools, however this guide is aimed more at those who are not experienced in this area and will only cover local run and setup. 

All tools below are opensource and free.
 
## Gitleaks [^2]

This tool by default checks all commits in the history, which is useful. 

### How 

1. Install docker [^1].
2. To run test on current directory:
    ```bash
    docker run -v $(pwd):/tmp/out \
    zricethezav/gitleaks:latest detect \
    --source="/tmp/out" -v  --no-color  > result.txt
    ```
3. This will fetch gitleaks from the remote registry, output will be in `result.txt`

## TruffleHog [^3]

By default checks only current branch, has a nice feature `-only-verified`, to get only true positive findings.

## How
1. Install docker [^1]
    ```bash
    docker run --rm -it -v "$(pwd):/out" \
    trufflesecurity/trufflehog:latest git file:///out | 
    sed -r 's/\x1b_[^\x1b]*\x1b[\]//g; s/\x1B\[[^m]*m//g' >
    result.txt
    ```
2. This will run TruffleHog in the current directory. The weird `sed` command just escapes ANSI colors.

## Do it yourself!

There is an infinite tradeoff between precision and variance. 
If you need more variance and don't mind more manual reviewing, you can try my tool [RegFinder](https://github.com/matejsmycka/regfinder), which is like grep, but recursive, and takes regex patterns from a file. 

1. Clone this [repository](https://github.com/matejsmycka/regfinder)
2. Run `./regfinder.elf -d your_app/ -f regex_dir/general.txt`

It is very easy to extend existing regex patterns. This tool is not feasible for automated pipeline, however it comes handy if you need to find either non-standard secret, or in something like security review, where you have more manual work is expected.

## Other tools

- https://www.gitguardian.com/
- https://github.com/michenriksen/gitrob
- https://github.com/eth0izzle/shhgit


## Resources 

[^1]: https://docs.docker.com/engine/install/
[^2]: https://github.com/gitleaks/gitleaks/
[^3]: https://github.com/trufflesecurity/trufflehog
[^4]: https://github.com/matejsmycka/regfinder
