---
title: 
  Map映射结构
tags: 
  [数据结构,映射]
categories:
  [数据结构]
date: 2019/11/25
---

## Map接口

```java
public interface Map<K,V>{
    void put(K key,V value);
    V remove(K key);
    boolean contains(K key);
    V get(K key);
    void set(K key,V newValue);
    int getSize();
    boolean isEmpty();
}
```

## LinkedListMap

```java
public class LinkedListMap<K,V> implements Map<K,V>{
    
    private class Node{
        public K key;
        public V value;
        public Node next;

        public Node(K key,V value,Node next){
            this.key=key;
            this.value=value;
            this.next=next;
        }

        public Node(K key){
            this(key,null,null);
        }

        public Node(){
            this(null,null,null);
        }

        @Override
        public String toString(){
            return key.toString()+" : "+value.toString();
        }
    }

    private Node dummyNode;

    private int size;

    public LinkedListMap(){
        dummyNode=new Node();
        size=0;
    }

    @Override
    public int getSize(){
        return size;
    }

    @Override
    public boolean isEmpty(){
        return size==0;
    }

    private Node getNode(K key){
        Node cur=dummyNode.next;
        while(cur!=null){
            if (cur.key.equals(key)) {
                return cur;
            }
            cur=cur.next;
        }
        return null;
    }

    @Override
    public boolean contains(K key){
        return getNode(key)==null;
    }

    @Override
    public V get(K key){
        Node node=getNode(key);
        return node==null?null:node.value;
    }

    @Override
    public void put(K key,V value){
        Node node = getNode(key);
        if(node==null){
            dummyNode.next=new Node(key,value,dummyNode.next);
            size++;
        }else{
            node.value=value;
        }
    }

    @Override
    public void set(K key,V value){
        Node node = getNode(key);
        if(contains(key)){
            node.value=value;
        }else{
            throw new IllegalArgumentException(key +" doesn't exist!");
        }
    }

    @Override
    public V remove(K key){
        Node prev=dummyNode;
        while(prev.next!=null){
            if (prev.next.key.equals(key)) {
                break;
            }
            prev=prev.next;
        }
        if (prev.next==null) {
            //其实可以和上面一样抛一个异常
            return null;
        }
        Node deleNode=prev.next;
        prev.next=deleNode.next;
        deleNode.next=null;
        size--;
        return deleNode.value;
    }
}
```

## BSTMap

```java
public class BSTMap<K extends Comparable,V> implements Map<K,V>{

    private class Node{
        public K key;
        public V value;
        public Node left;
        public Node right;
        public Node(K key,V value){
            this.key=key;
            this.value=value;
            left=null;
            right=null;
        }
    }  

    public Node root;

    private int size;

    public BSTMap(){
        root=null;
        size=0;
    }

    public int getSize(){
        return size;
    }

    public boolean isEmpty(){
        return size==0;
    }

    public void put(K key,V value){
        root=put(root,key,value);
    }

    //add元素后返回新的根节点
    private Node put(Node node,K key,V value){
        if (node == null) {
            size++;
            return new Node(key,value);
        }
        if(key.compareTo(node.key) < 0){
            node.left=put(node.left, key,value);
        }else if (key.compareTo(node.key) > 0) {
            node.right=put(node.right, key,value);   
        }else{
            //相等的情况
            node.value=value;
        }
        return node;
    }

    private Node getNode(Node node,K key){
        if (node==null) {
            return null;
        }
        int temp=node.key.compareTo(key);
        if (temp==0) {
            return node;
        }
        if (temp>0) { //node.key > key
            return getNode(node.left,key);
        }
        return getNode(node.right,key);
    }

    public boolean contains(K key){
        return getNode(root,key)!=null;
    }

    public V get(K key){
        Node node=getNode(root,key);
        return node==null?null:node.value;
    }

    public void set(K key,V newValue){
        Node node=getNode(root,key);
        if (node!=null) {
            node.value=newValue;
            return;
        }
        throw new IllegalArgumentException(key+ " doesn't exist");
    }

    private Node getMin(Node root){
        if (root.left==null) {
            return root;
        }
        return getMin(root.left);
    }

    private Node deleteMin(Node node){
        if (node.left==null) {
            return node.right;
        }
        node.left=deleteMin(node.left);
        return node;
    }

    public V remove(K key){
        Node node=getNode(root,key);
        if (node==null) {
            return null;
        }
        root=remove(node,key);
        size--;
        return node.value;
    }

    public Node remove(Node root,K key){
        if (key.compareTo(root.key)>0) { //key > root
            root.right=remove(root.right,key);
        }else if (key.compareTo(root.key)<0) {
            root.left=remove(root.left,key);
        }else{
            if (root.left==null) {
                return root.right;
            }
            if (root.right==null) {
                return root.left;
            }
            Node deleNode=root;
            root=getMin(root.right);
            root.right=deleteMin(deleNode.right);
            root.left=deleNode.left;
        }
        return root;
    }
}
```

## MapTest

```java
public class MapTest{
    public static void main(String[] args) {
        Map<Integer,Integer> map=new BSTMap<>();
        map.put(1,2);
        map.put(2,2);
        map.put(3,2);
        map.put(4,2);
        map.put(5,2);
        map.put(6,2);
        map.put(7,2);
        map.put(8,2);
        map.put(9,32131);
        map.put(11,2);
        System.out.println(map.contains(13)); //false
        System.out.println(map.getSize()); //10
        System.out.println(map.get(9)); //32131
        map.remove(9);
        System.out.println(map.get(9)); //null
        System.out.println(map.getSize()); //10
        map.set(11,1234567);
        System.out.println(map.get(11)); //1234567
    }
}
```

## HashMap

```java
import java.util.TreeMap;

public class HashTable<K,V>{
    private TreeMap<K,V>[] hashtable;
    private int M;
    private int size;

    private static final int upperTol=10;
    private static final int lowerTol=2;
    private static final int initCapacity=7;

    public HashTable(int M){
        this.M=M;
        size=0;
        hashtable=new TreeMap[M];
        for (int i=0;i<M;i++) {
            hashtable[i]=new TreeMap<>();
        }
    }

    public HashTable(){
        this(initCapacity);
    }

    private int hash(K key){
        return key.hashCode() & 0x7fffffff %M;
    }

    private int getSize(){
        return size;
    }

    public void add(K key,V value){
        TreeMap<K,V> map=hashtable[hash(key)];
        if(map.containsKey(key)){
            map.put(key,value);
        }else{
            map.put(key,value);
            size++;
            //扩容
            // size/M >=upperTol
            if (size>=upperTol*M) {
                resize(2*M);
            }
        }
    }

    public V remove(K key){
        TreeMap<K,V> map=hashtable[hash(key)];
        V ret=null;
        if(map.containsKey(key)){
            ret=map.remove(key);
            size--;
            if (size<lowerTol*M && M/2 >= initCapacity) {
                resize(M/2);
            }
        }
        return ret;
    }

    public void resize(int size){
        TreeMap<K,V> newHashTable=new TreeMap[size];
        for (int i=0;i<size;i++) {
            newHashTable[i]=new TreeMap<>();
        }
        int oldSize=M;
        this.M=size; //要先将M设置好,不然hash的值不对
        for (int i=0;i<oldSize;i++) {
            TreeMap<K,V> map=hashtable[i];
            for (K key:map.keySet()) {
                newHashTable[hash(key)].put(key,map.get(key));
            }
        }
        this.hashtable=newHashTable;
    }

    public void set(K key,V value){
        TreeMap<K,V> map=hashtable[hash(key)];
        if(!map.containsKey(key)){
            throw new IllegalArgumentException("key not exist!");
        }
        map.put(key,value);
    }

    public boolean contains(K key){
        return hashtable[hash(key)].containsKey(key);
    }

    public V get(K key){
        return hashtable[hash(key)].get(key);
    }
}
```

## 源码地址

[Github](https://github.com/imlgw/LeetCode/tree/master/tree/map)





