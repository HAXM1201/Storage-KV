### Trước khi cài Storage KV bạn phải cài Storage node vì nó liên quan mật thiết đến nhau

## Bắt đầu 

### 1. Cài đặt các phần phụ thuộc để xây dựng từ nguồn
   ```bash
   sudo apt-get update
   sudo apt-get install clang cmake build-essential
   ```

### 2. Cài go
   ```bash
   cd $HOME && \
   ver="1.22.0" && \
   wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
   sudo rm -rf /usr/local/go && \
   sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
   rm "go$ver.linux-amd64.tar.gz" && \
   echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
   source ~/.bash_profile && \
   go version
   ```

### 3. Cài rustup
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

### 4. Thêm thông số  vars
   Sau khi chạy lệnh bên dưới bạn sẽ phải điền thông tin vào  (http://STORAGE_NODE_IP:5678,http://STORAGE_NODE_IP:5679) , YOUR JSON RPC ENDPOINT (VALIDATOR_NODE_IP:8545) 
   hoặc enter nếu bạn đã sửa lệnh này trước đó
   
   ```bash
   read -p "Enter json-rpc: " BLOCKCHAIN_RPC_ENDPOINT && echo "Current json-rpc: $BLOCKCHAIN_RPC_ENDPOINT" &&
   read -p "Enter storage node urls: " ZGS_NODE && echo "Current storage node urls: $ZGS_NODE" &&
   echo 'export ZGS_LOG_SYNC_BLOCK="802"' >> ~/.bash_profile
   echo "export ZGS_NODE=\"$ZGS_NODE\"" >> ~/.bash_profile
   echo 'export LOG_CONTRACT_ADDRESS="0x8873cc79c5b3b5666535C825205C9a128B1D75F1"' >> ~/.bash_profile
   echo 'export MINE_CONTRACT="0x85F6722319538A805ED5733c5F4882d96F1C7384"' >> ~/.bash_profile
   echo "export BLOCKCHAIN_RPC_ENDPOINT=\"$BLOCKCHAIN_RPC_ENDPOINT\"" >> ~/.bash_profile
   
   source ~/.bash_profile
   
   echo -e "\n\033[31mCHECK YOUR STORAGE KV VARIABLES\033[0m\n\nZGS_NODE: $ZGS_NODE\nLOG_CONTRACT_ADDRESS: $LOG_CONTRACT_ADDRESS\nMINE_CONTRACT: $MINE_CONTRACT\nZGS_LOG_SYNC_BLOCK: $ZGS_LOG_SYNC_BLOCK\nBLOCKCHAIN_RPC_ENDPOINT: $BLOCKCHAIN_RPC_ENDPOINT\n\n" "\033[3m\"lets buidl together\" - Grand Valley\033[0m"
   ```

### 5. download binary
   thường một số guid sẽ bỏ qua lệnh này nhưng kiểu gì khi chạy nó cũng đòi
   ```bash
   apt install git
   ```
   ```bash
   cd $HOME
   git clone https://github.com/0glabs/0g-storage-kv.git
   cd $HOME/0g-storage-kv
   git fetch
   git checkout tags/v1.1.0-testnet
   git submodule update --init
   sudo apt install cargo
   ```
   then build it
   ```bash
   cargo build --release
   ```

### 6. copy a config_example.toml file
   ```bash
   cp /$HOME/0g-storage-kv/run/config_example.toml /$HOME/0g-storage-kv/run/config.toml
   ```

### 7. update storage kv configuration
   ```bash
   sed -i '
   s|^\s*#\?\s*rpc_enabled\s*=.*|rpc_enabled = true|
   s|^\s*#\?\s*rpc_listen_address\s*=.*|rpc_listen_address = "0.0.0.0:6789"|
   s|^\s*#\?\s*db_dir\s*=.*|db_dir = "db"|
   s|^\s*#\?\s*kv_db_dir\s*=.*|kv_db_dir = "kv.DB"|
   s|^\s*#\?\s*log_config_file\s*=.*|log_config_file = "log_config"|
   s|^\s*#\?\s*log_contract_address\s*=.*|log_contract_address = "'"$LOG_CONTRACT_ADDRESS"'"|
   s|^\s*#\?\s*zgs_node_urls\s*=.*|zgs_node_urls = "'"$ZGS_NODE"'"|
   s|^\s*#\?\s*mine_contract_address\s*=.*|mine_contract_address = "'"$MINE_CONTRACT"'"|
   s|^\s*#\?\s*log_sync_start_block_number\s*=.*|log_sync_start_block_number = '"$ZGS_LOG_SYNC_BLOCK"'|
   s|^\s*#\?\s*blockchain_rpc_endpoint\s*=.*|blockchain_rpc_endpoint = "'"$BLOCKCHAIN_RPC_ENDPOINT"'"|
   ' $HOME/0g-storage-kv/run/config.toml
   ```

### 8. create service
   ```bash
   sudo tee /etc/systemd/system/zgskv.service > /dev/null <<EOF
   [Unit]
   Description=ZGS KV
   After=network.target
   
   [Service]
   User=$USER
   WorkingDirectory=$HOME/0g-storage-kv/run
   ExecStart=$HOME/0g-storage-kv/target/release/zgs_kv --config $HOME/0g-storage-kv/run/config.toml
   Restart=on-failure
   RestartSec=10
   LimitNOFILE=65535
   
   [Install]
   WantedBy=multi-user.target
   EOF
   ```

### 9. Bắt đầu  node
   ```bash
   sudo systemctl daemon-reload && \
   sudo systemctl enable zgskv && \
   sudo systemctl start zgskv && \
   sudo systemctl status zgskv
   ```

### 10. check the logs
   ```bash
   sudo journalctl -u zgskv -fn 100 -o cat
   ```

### Nếu nó hiện như này thì chúc mừng bạn đã cài thành công :D

2024-07-18T08:34:05.348579Z  INFO stream::stream_manager::stream_replayer: tx 5682 is not in stream, skipped.
2024-07-18T08:34:05.349661Z  INFO stream::stream_manager::stream_data_fetcher: checking tx with sequence number 5679..
2024-07-18T08:34:05.350464Z  INFO stream::stream_manager::stream_replayer: checking tx with sequence number 5683..
2024-07-18T08:34:05.350498Z  INFO stream::stream_manager::stream_replayer: tx 5683 is not in stream, skipped.
2024-07-18T08:34:05.350519Z  INFO stream::stream_manager::stream_data_fetcher: tx 5679 is not in stream, skipped.
2024-07-18T08:34:05.351608Z  INFO stream::stream_manager::stream_replayer: checking tx with sequence number 5684..
2024-07-18T08:34:05.353291Z  INFO stream::stream_manager::stream_data_fetcher: checking tx with sequence number 5680..
2024-07-18T08:34:05.353405Z  INFO stream::stream_manager::stream_replayer: tx 5684 is not in stream, skipped.
2024-07-18T08:34:05.353761Z  INFO stream::stream_manager::stream_data_fetcher: tx 5680 is not in stream, skipped.
2024-07-18T08:34:05.355070Z  INFO stream::stream_manager::stream_replayer: checking tx with sequence number 5685..
2024-07-18T08:34:05.357026Z  INFO stream::stream_manager::stream_data_fetcher: checking tx with sequence number 5681..
2024-07-18T08:34:05.357091Z  INFO stream::stream_manager::stream_data_fetcher: tx 5681 is not in stream, skipped.
2024-07-18T08:34:05.357127Z  INFO stream::stream_manager::stream_replayer: tx 5685 is not in stream, skipped.
2024-07-18T08:34:05.357965Z  INFO stream::stream_manager::stream_data_fetcher: checking tx with sequence number 5682..
2024-07-18T08:34:05.358814Z  INFO stream::stream_manager::stream_replayer: checking tx with sequence number 5686..
2024-07-18T08:34:05.358850Z  INFO stream::stream_manager::stream_replayer: tx 5686 is not in stream, skipped.
2024-07-18T08:34:05.358874Z  INFO stream::stream_manager::stream_data_fetcher: tx 5682 is not in stream, skipped.
2024-07-18T08:34:05.359629Z  INFO stream::stream_manager::stream_replayer: checking tx with sequence number 5687..
2024-07-18T08:34:05.361141Z  INFO stream::stream_manager::stream_data_fetcher: checking tx with sequence number 5683..
2024-07-18T08:34:05.361190Z  INFO stream::stream_manager::stream_data_fetcher: tx 5683 is not in stream, skipped.



## delete the node
   ```bash
   sudo systemctl stop zgskv
   sudo systemctl disable zgskv
   sudo rm -rf /etc/systemd/system/zgskv.service
   sudo rm -rf 0g-storage-kv
   ```
