#

echo 'export SUBSPACE_NODENAME='NODE_NAME1 >> $HOME/.bash_profile
echo 'export SUBSPACE_WALLET='WALLET1 >> $HOME/.bash_profile
echo 'export SUBSPACE_ACCOUNT='ACCOUNT1 >> $HOME/.bash_profile
echo 'export SUBSPACE_PORT='PORT1 >> $HOME/.bash_profile
echo 'export SUBSPACE_WS_PORT='WS_PORT1 >> $HOME/.bash_profile
echo 'export SUBSPACE_PLOT_SIZE='PLOT_SIZE1 >> $HOME/.bash_profile
source ~/.bash_profile


apt update && apt upgrade -y
sudo apt install ocl-icd-opencl-dev libopencl-clang-dev libgomp1 -y
cd $HOME
wget -O subspace-node https://github.com/subspace/subspace/releases/download/gemini-2a-2022-sep-10/subspace-node-ubuntu-x86_64-gemini-2a-2022-sep-10
wget -O subspace-farmer https://github.com/subspace/subspace/releases/download/gemini-2a-2022-sep-10/subspace-farmer-ubuntu-x86_64-gemini-2a-2022-sep-10
chmod +x subspace*
source ~/.bash_profile
mkdir /usr/local/bin/$SUBSPACE_ACCOUNT /root/.local/share/$SUBSPACE_ACCOUNT
mv subspace* /usr/local/bin/$SUBSPACE_ACCOUNT

echo "[Unit]
Description=Subspace Node
After=network.target

[Service]
User=root
Type=simple
ExecStart=/usr/local/bin/$SUBSPACE_ACCOUNT/subspace-node --base-path /root/.local/share/$SUBSPACE_ACCOUNT/subspace-node --chain gemini-2a --pruning 1024 --validator --keep-blocks 1024 --port $SUBSPACE_PORT --ws-port $SUBSPACE_WS_PORT --name $SUBSPACE_NODENAME
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/$SUBSPACE_ACCOUNT.service



echo "[Unit]
Description=Subspaced Farm
After=network.target

[Service]
User=root
Type=simple
ExecStart=/usr/local/bin/$SUBSPACE_ACCOUNT/subspace-farmer --base-path /root/.local/share/$SUBSPACE_ACCOUNT/subspace-farmer farm --reward-address $SUBSPACE_WALLET --node-rpc-url ws://127.0.0.1:$SUBSPACE_WS_PORT --plot-size $SUBSPACE_PLOT_SIZE 
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/$SUBSPACE_ACCOUNT-farmer.service

mv $HOME/subspace* /etc/systemd/system/
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable $SUBSPACE_ACCOUNT $SUBSPACE_ACCOUNT-farmer
sudo systemctl restart $SUBSPACE_ACCOUNT

Ждем 10 секунд

sudo systemctl restart $SUBSPACE_ACCOUNT-farmer

journalctl -u subspace1 -f -o cat
journalctl -u subspace1-farmer -f -o cat


