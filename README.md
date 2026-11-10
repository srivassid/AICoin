# AICoin

AICoin (Useful-Work Coin) is a cryptocurrency mined on NVIDIA GPUs, where mining is matrix
multiplication, the same math AI models run. When your GPU runs paid AI jobs, that work also
counts toward finding blocks.

**This is a testnet.** Its coins have no value, and the chain may be reset.

`uwc-miner` is one download that mines and can run AI jobs. Nothing gets installed on your system:
everything lives in one folder, and your data in `~/.uwc-miner`.

## 1. Check your machine

| Need | Details |
|---|---|
| OS | Linux x86-64: Ubuntu 22.04 or newer, Debian 12 or newer. On Windows, use WSL2. |
| GPU | NVIDIA RTX 30, 40 or 50 series (tested on RTX 3080 and RTX 5090) |
| Driver | NVIDIA driver **580 or newer** |
| Disk | About 15 GB free: package 5 GB, AI model and its data 7 GB, download 3 GB |
| RAM | 8 GB to mine; 16 GB recommended for AI jobs |
| Network | Outbound access to `165.232.182.224` (ports 18690 and 18692) and `huggingface.co` |

You don't need CUDA, Python or anything else; it's all inside the package.

## 2. Install the NVIDIA driver

Skip this if `nvidia-smi` already shows driver 580 or newer.

**Ubuntu:**

```bash
nvidia-smi                       # shows "Driver Version: ..." if a driver is installed
sudo apt update
sudo apt install nvidia-driver-580
sudo reboot
```

**Windows (WSL2):**

1. In **Windows**, install the latest NVIDIA Game Ready or Studio driver (580 or newer). Don't
   install a driver inside WSL.
2. In PowerShell as administrator, run `wsl --install -d Ubuntu-24.04`, then restart Windows.
3. Open the Ubuntu terminal and run `nvidia-smi`. It should list your GPU. Run every later step in
   this terminal.

## 3. Download and unpack

Download both files into your home folder:

- [uwc-miner-testnet-20261110.1-linux-x64.tar.gz](https://pub-3395e2a5010d4d68a5ec8aeed07a73be.r2.dev/testnet/uwc-miner-testnet-20261110.1-linux-x64.tar.gz) (2.9 GB)
- [uwc-miner-testnet-20261110.1-linux-x64.tar.gz.sha256](https://pub-3395e2a5010d4d68a5ec8aeed07a73be.r2.dev/testnet/uwc-miner-testnet-20261110.1-linux-x64.tar.gz.sha256)

SHA-256: `a7088a3e3cb7ff3cd14eabf36b419d75fe105318c2bb238d66b39729393e3351`

```bash
cd ~
wget https://pub-3395e2a5010d4d68a5ec8aeed07a73be.r2.dev/testnet/uwc-miner-testnet-20261110.1-linux-x64.tar.gz
wget https://pub-3395e2a5010d4d68a5ec8aeed07a73be.r2.dev/testnet/uwc-miner-testnet-20261110.1-linux-x64.tar.gz.sha256
sha256sum -c uwc-miner-testnet-20261110.1-linux-x64.tar.gz.sha256    # must print: OK
tar xzf uwc-miner-testnet-20261110.1-linux-x64.tar.gz
rm uwc-miner-testnet-20261110.1-linux-x64.tar.gz                     # optional: frees 3 GB
cd uwc-miner-testnet-20261110.1
```

## 4. Check the GPU

```bash
./uwc-miner check
```

You should see your GPU, `driver: ok` and `cuda:0 kernels ok`. The first check compiles the GPU
code and takes a few seconds.

## 5. Start mining

```bash
./uwc-miner --jobs      # mining plus paid AI jobs (recommended)
./uwc-miner             # mining only
```

> **Back up your wallet.** The first run creates `~/.uwc-miner/testnet/wallet.json` and prints
> its address. That file is the only way to spend what you earn.

On the first run with `--jobs`, it also downloads the AI model (about 4 GB) and checks that it is
the exact version the network uses. That takes 5–10 minutes, once. Then it connects, catches up
with the chain, and prints a status line every 30 seconds:

```
height 1700, 1 peer | GPU0 167.6 TMAC/s | 3 blocks won | wallet 0 (+899 maturing) + jobs 0 | 0 jobs done
```

- **TMAC/s** is mining speed, in trillions of multiply-accumulate operations per second. Your
  share of blocks is your TMAC/s divided by the whole network's.
- **Maturing:** mined coins become spendable after 20 blocks.
- **Jobs balance:** with `--jobs`, it moves 100 mined coins there by itself. Jobs need a bond, so it
  takes work once those coins are in place.
- **GPU sharing:** while an AI job runs, normal mining on that GPU pauses, and the job itself mines.

Stop it with **Ctrl-C**. It shuts down cleanly in a few seconds.

## 6. Keep it running after you close the terminal

```bash
sudo apt install tmux
tmux new -s uwc
./uwc-miner --jobs
# detach: Ctrl-B, then D        reattach later: tmux attach -t uwc
```

## 7. Wallet commands

Run these while the miner runs, from a second terminal in the same folder.

```bash
./uwc-miner wallet             # address, spendable, maturing, jobs balance
./uwc-miner send 5 <address>   # pay 5 coins to someone
./uwc-miner deposit 10         # wallet -> jobs balance
./uwc-miner withdraw 10        # jobs balance -> wallet
```

## 8. Useful options

| Option | Effect |
|---|---|
| `--gpus 0,2` | Use only these GPUs (numbers as in `nvidia-smi`) |
| `--jobs-gpu 1` | Run AI jobs on GPU 1 instead of the first one |
| `--address <addr>` | Pay mined blocks to another address |
| `--reserve 0` | Don't move coins into the jobs balance automatically |
| `--nolisten` | Accept no incoming connections |

Forwarding TCP port 18690 on your router is optional. It lets other nodes connect to you, but
mining works without it.

## 9. Logs, updates, uninstall

- **Logs:** `~/.uwc-miner/testnet/logs/`: `node.log`, `gateway.log`, `miner-gpu0.log`,
  `jobs.log`.
- **Update:** stop the miner, unpack the new version and run it from the new folder. Your wallet
  and data in `~/.uwc-miner/` stay. From block 2700 the testnet runs Rules 4 (Llama and Mistral
  models): it needs `testnet-20261110.1` or newer, and older packages stop following the chain
  there.
- **Uninstall:** delete the package folder and `~/.uwc-miner/`. Back up `wallet.json` first if you
  want to keep the coins.

## 10. Troubleshooting

| You see | What to do |
|---|---|
| `driver ... is too old` | Install driver 580 or newer (step 2). |
| `no NVIDIA GPU found` | `nvidia-smi` must work first. On WSL, update the Windows driver. |
| `GLIBC_2.33 not found` | Your Linux is too old; use Ubuntu 22.04 or newer. |
| `0 peers` for minutes | Your network blocks outbound port 18690 to `165.232.182.224`. |
| `another uwc-miner is already running` | It's already running, perhaps in tmux: `tmux attach -t uwc`. |
| `cannot bond` in `jobs.log` | Wait for mined coins to mature (20 blocks). The miner tops up the jobs balance by itself. |
| Low TMAC/s at start | Normal for the first minute, and while the AI model loads on the CPU. |
