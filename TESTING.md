# Testing AICoin: a note for testers

Thanks for helping test AICoin. This page says what we'd like you to try, what to expect, and how
to tell us when something goes wrong. Installing is covered in the [README](README.md).

## Before you start

- **This is a testnet.** The coins have no value and can't be bought or sold. The chain may be
  paused or reset, and wallets may be wiped. Don't use a wallet you care about.
- **It runs as your normal user.** Nothing is installed system-wide and it never asks for `sudo`.
  To remove it, delete the package folder and `~/.uwc-miner/`.
- **Your node is visible to the network.** It connects to the seed node `165.232.182.224`, and
  other nodes can see your IP address. Use `--nolisten` if you don't want incoming connections.
- **AI job inputs are public.** Everything sent to the network as a job (prompts, texts, files in
  a batch) is stored on a public data service so that others can check the work. Only send text
  you're happy for anyone to read: no personal data, passwords or private documents.
- **Expect bugs.** Finding them is the point. Please report them even if they seem small.

## What to try

Work down the list as far as you like. The first three are the most useful.

1. **Install and check.** Follow the README up to `./uwc-miner check`. Tell us if any step was
   unclear, slow or failed.
2. **Mine with AI jobs for an hour.** Run `./uwc-miner --jobs`. Within a few minutes you should
   see the height rising, then `blocks won` going up. Mined coins mature after 20 blocks.
3. **Leave it running for a day.** Use tmux (README step 6). Long runs find the bugs short ones
   don't: a miner that stops, falls behind the chain, or fills the GPU's memory.
4. **Stop and start it.** Press Ctrl-C, wait for it to exit, and start it again. Also try
   rebooting the machine mid-run, then starting it again (it doesn't start by itself). It should
   catch up with the chain and keep the same wallet.
5. **Send coins to another tester.** Swap addresses (`./uwc-miner wallet` shows yours) and try
   `./uwc-miner send 5 <their address>`. Check that it arrives.
6. **Use the jobs page:** <https://165-232-182-224.sslip.io/>. Create a key there (no signup
   needed); it comes with 100 testnet coins. Try:
   - **Chat**, with each model in the list;
   - **Complete text** with Llama 3.1 8B (a base model: give it the start of a sentence);
   - **Embeddings** (Qwen3);
   - **Batch**: upload a small text file, one input per line, and download the results;
   - **My jobs**, to watch a request's trust go from *committed* to *verified* to *final*.

   An answer usually takes a few minutes, and reaching *final* takes about 5 hours. That's how
   long the network allows for challenges.

## What "working" looks like

- `./uwc-miner check` prints your GPU, `driver: ok` and `kernels ok`.
- The status line, every 30 seconds, shows a rising `height` and `1 peer` or more. On a small
  network, `blocks won` goes up a few times an hour.
- With `--jobs`, `jobs.log` shows `claim`, `accept`, `commit` and `reveal` lines now and then.
  Paid work is limited on the testnet right now, so long quiet spells are normal.
- `cannot bond` in `jobs.log` right after starting is normal. It goes away once mined coins mature.

## How to report a problem

Open an issue: <https://github.com/srivassid/AICoin/issues>. Please include:

- **Package version:** the folder name, or `cat VERSION` in it (e.g. `testnet-20261111.2`).
- **Your machine:** GPU model, the first lines of `nvidia-smi`, and your OS (Ubuntu version, or
  Windows with WSL2).
- **What you did** and **what happened**, with the exact error text if there was one.
- **When it happened**, with your time zone.
- **Logs:** the last 200 lines of each log around the time it happened:

  ```bash
  cd ~/.uwc-miner/testnet/logs
  for f in node.log jobs.log miner-gpu0.log gateway.log; do echo "== $f"; tail -n 200 "$f"; done > ~/uwc-report.txt
  ```

  Attach `~/uwc-report.txt` to the issue. Look through it first if you're unsure what it contains.
- For a jobs-page problem: the request ID shown in **My jobs**, and your browser.

**Never post `wallet.json` or your API key.** We will never ask for them.

## Known limits right now

- **Little paid work.** With `--jobs`, your GPU mostly mines and only sometimes computes a job.
- **The miner computes Qwen3 jobs only.** The Llama and Mistral jobs on the jobs page are
  computed by the project's own machines.
- **One seed node.** If `165.232.182.224` is down, new miners can't find the network, and a
  running miner that reached others only through it loses its peers until it's back.
- **Updates.** When the network's rules change, older packages stop following the chain at a
  set block. The README always links the current package. If your miner stops gaining height
  while others continue, check there for a new version.

Thanks again: every report makes the next version better.
