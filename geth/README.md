# Geth
```bash
nohup ./geth --allow-insecure-unlock --cache=512 --datadir=./mainnet --rpc --rpcaddr="0.0.0.0" --rpcapi="eth,net,web3,personal" --ws --wsorigins="*" --wsapi="net,web3,eth,shh,personal" >>./main.log 2>&1 &
```
