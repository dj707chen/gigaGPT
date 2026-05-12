This project uses the requirements.txt to manage dependencies,
I'm not familiar with it, so Gooled

    Tell me how to use PYTHON-SETUP.md, setup.py and requirements.txt in Python project.

```shell
uv run --no-project --with datasets==2.19.1 --with tiktoken==0.7.0 --with tqdm==4.66.1 --with numpy==1.24.4 python data/openwebtext/prepare.py

uv run python data/openwebtext/prepare.py

uv sync
# Resolved 85 packages in 347ms
# error: Distribution `cerebras-pytorch==2.5.0 @ registry+https://pypi.org/simple` can't be installed because it doesn't have a source distribution or wheel for the current platform
```

## Need to work on Linux system on x86_64!

Due the above error, need to try in Linux shell:
```shell
source .venv/bin/activate
export PATH=$PATH:/Users/weiping/.antigravity/antigravity/bin:/Users/weiping/.bun/bin:/Users/weiping/.nvm/versions/node/v24.12.0/bin:/Users/weiping/.sdkman/candidates/scala/current/bin:/Users/weiping/.sdkman/candidates/sbt/current/bin:/Users/weiping/.sdkman/candidates/java/current/bin:/Applications/PyCharm.app/Contents/MacOS:/Users/weiping/.krew/bin:/Users/weiping/w1/environments/scripts/debugging:/Users/weiping/tools/helix-24.07-x86_64-macos:/Users/weiping/repoMy/fs2-kafkaLab/scripts:/Users/weiping/dev/groovy-3.0.9/bin:/Users/weiping/repoOSS/redis/src:/Users/weiping/.local/bin:/Users/weiping/go/bin:/Users/weiping/tools:/Users/weiping/bin:/Applications/RustRover.app/Contents/MacOS:/opt/homebrew/opt/postgresql@15/bin:/usr/local/Cellar/podman-compose/1.0.6/bin:/usr/local/Cellar/podman/5.1.1/bin:/usr/libexec:/opt/homebrew/Cellar/bash/5.3.3/bin:/Users/weiping/.jenv/shims:/Users/weiping/.jenv/bin:/bin:.:./node_modules/.bin:/Users/weiping/dev/confluent-6.2.1/bin:/usr/local/opt/scala@2.13/bin:/bin:/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/local/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/appleinternal/bin:/opt/pmk/env/global/bin:/usr/local/go/bin:/Users/weiping/.cargo/bin:/Applications/iTerm.app/Contents/Resources/utilities
```

But this does not work, next considering a Linux laptop:
- best laptop running x86_64: https://www.google.com/search?q=best+laptop+running+x86_64&oq=best+laptop+running+x86_64&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIHCAEQIRigATIHCAIQIRigATIHCAMQIRigATIHCAQQIRigATIHCAUQIRigATIHCAYQIRiPAjIHCAcQIRiPAtIBCDcxMTBqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8
- Install Linux on Windows: https://www.google.com/search?q=install++linux+on+Lenovo+ThinkPad+x1&sca_esv=80bb2a0df4fb4cf7&sxsrf=ANbL-n5P0o7zMMG-N2y0Ufy_Bgt1xPdBWQ%3A1778037564000&ei=O7P6abPqPKqvptQPyvflqQw&biw=1689&bih=880&ved=0ahUKEwjz85iL2qOUAxWql4kEHcp7OcUQ4dUDCBM&uact=5&oq=install++linux+on+Lenovo+ThinkPad+x1&gs_lp=Egxnd3Mtd2l6LXNlcnAiJGluc3RhbGwgIGxpbnV4IG9uIExlbm92byBUaGlua1BhZCB4MTIFECEYoAEyBRAhGKABMgUQIRigATIFECEYoAEyBRAhGKABMgUQIRirAkiqhgFQjAVYvIIBcAJ4AZABAJgBXqABxQqqAQIxObgBA8gBAPgBAfgBApgCFaACpwvCAgoQABhHGNYEGLADwgILEAAYgAQYigUYkQLCAgYQABgHGB7CAgUQABiABMICChAAGIAEGBQYhwLCAgYQABgWGB7CAgsQABiABBiKBRiGA8ICBRAAGO8FwgIIEAAYgAQYogTCAggQABgWGB4YCpgDAIgGAZAGCJIHAjIxoAeeZLIHAjE5uAedC8IHBjAuMTMuOMgHO4AIAQ&sclient=gws-wiz-serp
