# Create Docker Image #3

1. Setup a separate branch: `git checkout -b ci/docker`. The -b flag creates a new branch and switches to it immediately.
2. Copy [Gradle Dockerfile](https://codefresh.io/docs/docs/example-catalog/ci-examples/gradle/).
3. Run `docker build -t admin_backend .`. -t is the tag. We can name it whatever. For convenience, we went with admin_backend. The dot means in the current directory. This is commonplace for all Unix commands (i.e. cp file_name.txt .).
4. Build failed! The version used in the initial Dockerfile are not supported by the Gradle packages. Look up [Gradle Docker Images](https://hub.docker.com/_/gradle). In gradle.yml, the version is JDK17. Let's look for a version that supports JDK17. Oh, here's one: `8.7.0-jdk17`. It is assumed alpine is default, so no need to find a specific one. Run build command again.
5. Build failed! Lots of Java call errors. Test cases seem to not be loading. Turns out at this moment, there is no database. We can ignore this by using the -x flag which excludes tasks. In our case, ignore the test cases. So the build should run...
6. Build successful! Let's push. But wait, the local is not set to the git provider. SO, let's establish the connection: `git push --set-upstream origin ci/docker`. We can create a pull request but not now.
7. Ok, let's add GitHub actions. First, in blueprint_admin repo, let's copy `.github/workflows/ci.yml`. Since we are working on separate branch, let's add our branch `["main", "ci/docker]`.
8. Change username to my username. Now we add secrets. The first is personal GitHub access token. Go to Profile >> Settings >> Developer settings >> Personal Access Tokens >> Tokens (classic) >> Generate New token (classic). Check `write:packages` permissions. Generate token. Copy Personal Access Token.
9. Now let's hook that Personal Access Token to Repository Secrets. First, get admin access to the repo. Then, `Repo Settings >> Security >> Secrets and variables >> Actions >> Repository Secrets >> New repository secret >> Name and add private Secret`. You NEED to change the password field in the new .yml file to be the name of the new repository secret you just put in.
10. Change tags in the .yml file to be the appropriate file path for the repo `ghcr.io/stevensblueprint/blueprint_admin_backend`. ghcr.io is GitHub's Container Registry.
11. Create PR, add reviewer to review. When they approve and build passes, merge into main.

## CONGRATS! IT'S OVER
