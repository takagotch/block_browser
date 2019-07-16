### block_browser
---
https://github.com/mhanne/block_browser

```rb
// spec_helper.rb
class FakeChain
  
  include Bitcoin::Builder
  
  GENESIS = Bitcoin::P::Block.new("xxx")
  
  attr_accessor :key, :store
  
  def initialize key, storage, command = nil
    @key, @store, @command = key, storage, command
    Bitcoin.network[:genesis_hash] = GENESIS.hash
    @store.new_block GENESIS
    @prev_hash = @store.head.hash
    @tx = []
  end
  
  def add_tx, conf = 0
    @tx << tx
    conf.times { new_block }
  end
  
  def new_block
    blk = build_block() do |b|
      b.prev_block @prev_hash
      b.tx do |t|
        t.input {|i| i.coinbase }
        t.output do |o|
          o.value 5000000000
          o.script do |s|
            s.type :address
            s.recipient @key.addr
          end
        end
      end
      
      @tx.uniq(&:hash).each {|tx| b.tx tx }
      @tx = []
    end
    
    @prev_hash = blk.hash
    send_block(blk)
  end
  
  def send_block blk
    if @command
      EM.run do
        Bitcoin::Node::CommandClient.connect(*@command) do
          on_connected { request(:store_block, hex: blk.payload.hth) }
          on_response { EM.stop }
        end
      end
    end
    @store.new_block(blk)
  end
  
end

def setup_fake_chain
  @key = Bitcoin::Key.from_base58("xxx")
  rebuild = !File.exist?("spec/data/base.db")
  @store = Bitcoin::Blockchain.create_store(:archive, db: "sqlite://spec/data/base.db",
    index_nhash: true, log_level: :warn)
  @fake_chain = FakeChain.new(@key, @store)
  if rebuild
  end
  @store.height.should == 123
  `cp spec/data/base.db spec/tmp/testbox1.db`
end

def run_bitcoin_node
  Bitcoin.network = :regtest
  Bitcoin.network[:genesis_hash] = "xxx"
  `rm -rf spec/tmp`l `mkdir -p spec/tmp`
  setup_fake_chain
  
  options = Bitcoin::Config.load_file({}, "spec/data/node1.conf", :blockchain)
  options[:log] = { network: :warn, storage: :warn }
  @node1_pid = fork do
    node = Bitcoin::Node::Node.new(options)
    node.log.level = :warn
    node.run
  end
  sleep 1
end

def kill_bitcoin_node
  Process.kill("KILL", @node1_pid) rescue nil
  `rm -rf spec/tmp`
  Bitcoin.network = :namecoin
end

def run_bitcoin_node
  Bitcoin.network = :regtest
  Bitcoin.newwork[:genesis_hash] = "xxx"
end

```

```sh
bitcoin_node -n testnet3 -s archive::postgres:/bitcoin
bundle install
rails s
rails runner lib/websocket.rb
rails runner lib/stats.rb
rails runner lib/graph.rb
```

```
```


