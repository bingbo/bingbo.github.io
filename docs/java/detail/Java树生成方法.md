数据库中我们常在一个表中设计树结构的数据，如id,name,pid等，显示的时候需要以树型目录结构显示，以下是用java List生成树的方法

### 树节点定义

```java
import java.util.List;

public class Node implements java.io.Serializable {
    private static final long serialVersionUID = -1L;

    private int id;

    private int parentId;

    private Node parent;

    private List<Node> children;

    private String name;

    private int level;

    private int sort;

    private int rootId;

    private String type;

    private boolean isLeaf;

    private String description;

    public Node() {
        super();
    }

    public Node(int id, int parentId, String name) {
        super();
        this.id = id;
        this.parentId = parentId;
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public Node getParent() {
        return parent;
    }

    public void setParent(Node parent) {
        this.parent = parent;
    }

    public int getParentId() {
        return parentId;
    }

    public void setParentId(int parentId) {
        this.parentId = parentId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getLevel() {
        return level;
    }

    public void setLevel(int level) {
        this.level = level;
    }

    public String getType() {
        return type;
    }

    public List<Node> getChildren() {
        return children;
    }

    public void setChildren(List<Node> children) {
        this.children = children;
    }

    public void setType(String type) {
        this.type = type;
    }

    public boolean isLeaf() {
        return isLeaf;
    }

    public void setLeaf(boolean isLeaf) {
        this.isLeaf = isLeaf;
    }

    public int getSort() {
        return sort;
    }

    public void setSort(int sort) {
        this.sort = sort;
    }

    public int getRootId() {
        return rootId;
    }

    public void setRootId(int rootId) {
        this.rootId = rootId;
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + id;
        result = prime * result + parentId;
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        Node other = (Node) obj;
        if (id != other.id)
            return false;
        if (parentId != other.parentId)
            return false;
        return true;
    }

    @Override
    public String toString() {
        return "Node {id=" + id + ", parentId=" + parentId + ", children="
                + children + ", name=" + name + ", level =" + level + "}";
    }
}
```

### 树生成工具类

```java
import java.util.ArrayList;
import java.util.List;

import org.apache.commons.collections.CollectionUtils;

public class TreeBuilder {

    @SuppressWarnings("unchecked")
    private List<Node> buildListToTree(List<Node> dirs) {
        List<Node> roots = findRoots(dirs);
        List<Node> notRoots = (List<Node>) CollectionUtils
                .subtract(dirs, roots);
        for (Node root : roots) {
            root.setChildren(findChildren(root, notRoots));
        }
        return roots;
    }

    public List<Node> findRoots(List<Node> allNodes) {
        List<Node> results = new ArrayList<Node>();
        for (Node node : allNodes) {
            boolean isRoot = true;
            for (Node comparedOne : allNodes) {
                if (node.getParentId() == comparedOne.getId()) {
                    isRoot = false;
                    break;
                }
            }
            if (isRoot) {
                node.setLevel(0);
                results.add(node);
                node.setRootId(node.getId());
            }
        }
        return results;
    }

    @SuppressWarnings("unchecked")
    private List<Node> findChildren(Node root, List<Node> allNodes) {
        List<Node> children = new ArrayList<Node>();

        for (Node comparedOne : allNodes) {
            if (comparedOne.getParentId() == root.getId()) {
                comparedOne.setParent(root);
                comparedOne.setLevel(root.getLevel() + 1);
                children.add(comparedOne);
            }
        }
        List<Node> notChildren = (List<Node>) CollectionUtils.subtract(allNodes, children);
        for (Node child : children) {
            List<Node> tmpChildren = findChildren(child, notChildren);
            if (tmpChildren == null || tmpChildren.size() < 1) {
                child.setLeaf(true);
            } else {
                child.setLeaf(false);
            }
            child.setChildren(tmpChildren);
        }
        return children;
    }

    public static void main(String[] args) {
        TreeBuilder tb = new TreeBuilder();
        List<Node> allNodes = new ArrayList<Node>();
        allNodes.add(new Node(1, 0, "节点1"));
        allNodes.add(new Node(2, 0, "节点2"));
        allNodes.add(new Node(3, 0, "节点3"));
        allNodes.add(new Node(4, 1, "节点4"));
        allNodes.add(new Node(5, 1, "节点5"));
        allNodes.add(new Node(6, 1, "节点6"));
        allNodes.add(new Node(7, 4, "节点7"));
        allNodes.add(new Node(8, 4, "节点8"));
        allNodes.add(new Node(9, 5, "节点9"));
        allNodes.add(new Node(10, 100, "节点10"));
        List<Node> roots = tb.buildListToTree(allNodes);
        for (Node n : roots) {
            System.out.println(n);
        }

    }
}
```

