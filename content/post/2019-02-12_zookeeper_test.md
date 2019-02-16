---
title: "2019 02 12_zookeeper_test"
date: 2019-02-12T22:45:55+08:00
draft: false
title: "zookeeper学习(1)小例子"
categories: [zookeeper]
---

watch小例子：

```java
public class ConnectionWatcher implements Watcher {
    private CountDownLatch connectedSignal = new CountDownLatch(1);
    private static final int SESSION_TIMEOUT = 5000;
    protected ZooKeeper zk;

    public void connect(String hosts) throws InterruptedException, IOException {
        zk = new ZooKeeper(hosts, SESSION_TIMEOUT, this);
        connectedSignal.await();
    }

    public void process(WatchedEvent event) {
        if(event.getState() == Event.KeeperState.SyncConnected) {
            connectedSignal.countDown();
        }
    }

    public void close() throws InterruptedException {
        zk.close();
    }
}
```

列出节点的子节点：

```java
public class HelloZookeeper extends ConnectionWatcher {

    public void list(String groupName) throws KeeperException, InterruptedException{
        String path ="/"+groupName;
        try {
            List<String> children = zk.getChildren(path, false);
            if(children.isEmpty()){
                System.out.printf("No memebers in group %s\n",groupName);
                System.exit(1);
            }
            for(String child : children){
                System.out.printf("group = %s, child = %s\n", groupName, child);
            }
        } catch (KeeperException.NoNodeException e) {
            System.out.printf("Group %s does not exist!\n", groupName);
            System.exit(1);
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        HelloZookeeper helloZookeeper = new HelloZookeeper();
        helloZookeeper.connect(args[0]);
        helloZookeeper.list(args[1]);
        helloZookeeper.close();
    }

}
```

![](http://ww1.sinaimg.cn/large/6120fe13gy1g08p17ew2vj21ll1akq9m.jpg)


