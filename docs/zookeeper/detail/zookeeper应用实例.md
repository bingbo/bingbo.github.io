## zookeeper client实例

### 添加依赖

```xml
<!--zookeeper-->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
    <exclusions>
        <exclusion>
            <artifactId>slf4j-api</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
        <exclusion>
            <artifactId>log4j</artifactId>
            <groupId>log4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

### 调用方式

```java
package com.ibingbo.zookeeper;

import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;
import java.util.concurrent.CountDownLatch;

/**
 * Created by bing on 2017/3/9.
 */
public class Test implements Watcher {

    private static final Logger LOGGER = LoggerFactory.getLogger(Test.class);

    private static final int TIMEOUT = 3000;
    private static final String HOST = "127.0.0.1:2181";
    private static CountDownLatch latch = new CountDownLatch(1);
    private static ZooKeeper zooKeeper = null;
    private static ZooKeeper zooKeeper1 = null;

    public static void main(String[] args) {
        try {
            zooKeeper = new ZooKeeper(HOST, TIMEOUT, new Test());
            zooKeeper1 = new ZooKeeper(HOST, TIMEOUT, null);
            //为该会话添加权限控制
            zooKeeper.addAuthInfo("digest","bill:123".getBytes());

            //获取zookeeper连接状态
            System.out.println("state: " + zooKeeper.getState());

            try {
                latch.await();
            } catch (InterruptedException e) {
                System.out.println("zookeeper session established.");
            }

            //获取这次连接的会话ID及密码
            System.out.println("sessionId: " + zooKeeper.getSessionId());
            System.out.println("sessionPasswd: " + zooKeeper.getSessionPasswd());

            //判断是否存在某个节点及创建节点
            if (zooKeeper.exists("/zk_test", false) == null) {
                System.out.println("have no zk_test node");
                zooKeeper.create("/zk_test", "mydata".getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT, new MyStringCallback(), "my contexxt");
            }


            //获取指定路径下的子节点
            List<String> children = zooKeeper.getChildren("/", new Test());
            LOGGER.info("all children: {}", children);

            //获取指定节点下的数据
            System.out.println("-------check node--------");
            Stat stat = new Stat();
            System.out.println(new String(zooKeeper.getData("/zk_test", true, stat)));
            LOGGER.error("czxid: {} mzxic: {} version: {}", stat.getCzxid(), stat.getMzxid(), stat.getVersion());

            //试图访问有权限的数据，没有权限会抛出异常KeeperErrorCode = NoAuth for /zk_test
            //LOGGER.info("********************");
            //zooKeeper1.getData("/zk_test", false, null);

            //更新指定节点下数据
            System.out.println("-------update node data-------");
            stat = zooKeeper.setData("/zk_test", "update data".getBytes(), -1);
            LOGGER.error("czxid: {} mzxic: {} version: {}", stat.getCzxid(), stat.getMzxid(), stat.getVersion());

            System.out.println("-------check node--------");
            zooKeeper.getData("/zk_test", true, new MyDataCallback(), null);

            //删除指定节点下的子节点，只能先删除子节点后，才能删除父节点
            System.out.println("--------delete node-------");
            zooKeeper.delete("/zk_test", -1);

            System.out.println("-----------check node is deleted-----");
            System.out.println("node status: " + zooKeeper.exists("/zk_test", false));

            int i=1;
            while (i<100) {
                zooKeeper1.setData("/zkclient", ("hello,world "+i).getBytes(),-1);
                i++;
                Thread.sleep(1000);
            }
            zooKeeper.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 监视节点变化的回调处理
     * @param watchedEvent
     */
    public void process(WatchedEvent watchedEvent) {
        LOGGER.info("watcher receive: {}", watchedEvent);
        try {
            if (watchedEvent.getState() == Event.KeeperState.SyncConnected) {
                if (watchedEvent.getType() == Event.EventType.None && null == watchedEvent.getPath()) {
                    latch.countDown();
                } else if (watchedEvent.getType() == Event.EventType.NodeChildrenChanged) {
                    LOGGER.info("reget child: {}", zooKeeper.getChildren(watchedEvent.getPath(), true));
                } else if (watchedEvent.getType() == Event.EventType.NodeDataChanged) {
                    Stat stat = new Stat();
                    LOGGER.info("reget data:{}", zooKeeper.getData(watchedEvent.getPath(), true, stat));
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 数据变化回调
     */
    static class MyDataCallback implements AsyncCallback.DataCallback {
        public void processResult(int resultCode, String path, Object context, byte[] data, Stat stat) {
            LOGGER.info("resultCode:{} path:{} context:{} data:{} stat:{}", resultCode, path, context, new String(data), stat);
        }
    }

    /**
     * 节点路径变化回调
     */
    static class MyStringCallback implements AsyncCallback.StringCallback {
        public void processResult(int resultCode, String path, Object context, String nodeName) {
            LOGGER.info("create path result: resultCode-{} , path-{}, context-{}, real path name-{}", resultCode, path, context, nodeName);
        }
    }

}
```

## zkclient实例

### 添加依赖

```xml
<!--zkclient-->
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.10</version>
    <exclusions>
        <exclusion>
            <artifactId>slf4j-api</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

### 调用方式

```java
package com.ibingbo.zookeeper;

import org.I0Itec.zkclient.IZkChildListener;
import org.I0Itec.zkclient.IZkDataListener;
import org.I0Itec.zkclient.IZkStateListener;
import org.I0Itec.zkclient.ZkClient;
import org.apache.zookeeper.Watcher;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;

/**
 * Created by bing on 2017/3/23.
 */
public class ZkClientTest {
    private static final Logger LOGGER = LoggerFactory.getLogger(ZkClientTest.class);

    public static void main(String[] args) throws InterruptedException {
        ZkClient zkClient = new ZkClient("127.0.0.1:2181", 5000);
        LOGGER.warn("zookeeper session established");
        String path = "/zkclient";

        zkClient.subscribeChildChanges("/zkclient", new IZkChildListener() {
            public void handleChildChange(String parentPath, List<String> currentChildren) throws Exception {
                LOGGER.warn("parent: {}, children: {}", parentPath, currentChildren);
            }
        });
        zkClient.subscribeDataChanges(path, new IZkDataListener() {
            public void handleDataChange(String s, Object o) throws Exception {
                LOGGER.info("path:{},new data:{}", s, o);
            }

            public void handleDataDeleted(String s) throws Exception {
                LOGGER.info("path:{} 's data deleted", s);
            }
        });
        zkClient.subscribeStateChanges(new IZkStateListener() {
            public void handleStateChanged(Watcher.Event.KeeperState keeperState) throws Exception {
                LOGGER.warn("state:{}", keeperState);
            }

            public void handleNewSession() throws Exception {
                LOGGER.warn("new session");
            }

            public void handleSessionEstablishmentError(Throwable throwable) throws Exception {
                LOGGER.warn("new session error",throwable);
            }
        });

        zkClient.delete(path+"/c1");
        zkClient.delete(path);

        zkClient.createPersistent(path);
        LOGGER.warn("children:{}",zkClient.getChildren(path));
        Thread.sleep(1000);

        zkClient.createPersistent("/zkclient/c1", true);
        LOGGER.warn("children:{}",zkClient.getChildren(path));
        Thread.sleep(1000);

        LOGGER.warn("path:{},data:{}", path, zkClient.readData(path));
        zkClient.writeData(path, "hello,bill");
        Thread.sleep(Integer.MAX_VALUE);

        //zkClient.delete(path+"/c1");
        //zkClient.delete(path);



    }
}
```

## curator zk client实例

### 添加依赖

```xml
<!--curator zk client
            Curator 2.x.x - compatible with both ZooKeeper 3.4.x and ZooKeeper 3.5.x
Curator 3.x.x - compatible only with ZooKeeper 3.5.x and includes support for new features such as dynamic
-->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.9.0</version>
</dependency>
```

### 调用方式

```java
package com.ibingbo.zookeeper;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.framework.api.CuratorListener;
import org.apache.curator.framework.recipes.cache.*;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.Watcher;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Created by bing on 2017/3/23.
 */
public class CuratorZkClientTest {
    private static final Logger LOGGER = LoggerFactory.getLogger(CuratorZkClientTest.class);

    private static CuratorFramework client = CuratorFrameworkFactory.newClient("127.0.0.1:2181", 5000, 3000, new ExponentialBackoffRetry(1000, 3));


    public static void main(String[] args) throws Exception {
        final String path = "/curator_zk";
        client.start();

        client.delete().forPath(path);
        client.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath(path);
        LOGGER.warn("nodes:{}",client.getChildren().forPath("/"));

        //节点数据变更监控
        final NodeCache nodeCache = new NodeCache(client, path, false);
        nodeCache.getListenable().addListener(new NodeCacheListener() {
            public void nodeChanged() throws Exception {
                LOGGER.warn("nodes:{}", nodeCache.getCurrentData().getData());
            }
        });

        //节点变更监控
        final PathChildrenCache pathChildrenCache = new PathChildrenCache(client, path, true);
        pathChildrenCache.start();
        pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
            public void childEvent(CuratorFramework curatorFramework, PathChildrenCacheEvent pathChildrenCacheEvent) throws Exception {
                switch (pathChildrenCacheEvent.getType()) {
                    case CHILD_ADDED:
                        LOGGER.warn("child added in {},children is {}", pathChildrenCacheEvent.getData().getPath(), curatorFramework.getChildren().forPath(path));
                        break;
                    case CHILD_REMOVED:
                        LOGGER.warn("child removed in {},children is {}", pathChildrenCacheEvent.getData().getPath(), curatorFramework.getChildren().forPath(path));
                        break;
                    case CHILD_UPDATED:
                        LOGGER.warn("child updated in {},children is {}", pathChildrenCacheEvent.getData().getPath(), curatorFramework.getChildren().forPath(path));
                        break;

                }
            }
        });

        client.setData().forPath(path, "hello,world".getBytes());
        byte[] data = client.getData().forPath(path);
        LOGGER.warn("node data:{}", new String(data));

        for (int i=0;i<10;i++) {
            client.create().withMode(CreateMode.PERSISTENT).forPath(path + "/node_" + i);
            Thread.sleep(1000);
        }
        for (int i=0;i<10;i++) {
            client.delete().forPath(path + "/node_" + i);
            Thread.sleep(1000);
        }

    }
}
```
