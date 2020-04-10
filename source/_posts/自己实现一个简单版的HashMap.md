title: 自己实现一个简单版的HashMap
tags:
  - HashMap
categories:
  - Dev
  - Java
date: 2019-07-26 11:54:00

---
<!-- more -->

## HashMap简介

HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。
HashMap 继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口。
HashMap 的实现不是同步的，这意味着它不是线程安全的。它的key、value都可以为null。此外，HashMap中的映射不是有序的。


HashMap 的实例有两个参数影响其性能：“初始容量” 和 “加载因子”。容量 是哈希表中桶的数量，初始容量 只是哈希表在创建时的容量。加载因子 是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行 rehash 操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。
通常，默认加载因子是 0.75, 这是在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数 HashMap 类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少 rehash 操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生 rehash 操作。

## 简单版，只实现put和get
```
public class MyHashMap<K, V> {
    private static int default_length = 16;
    private MyEntry<K, V>[] entries;

    public MyHashMap() {
        super();
        entries = new MyEntry[default_length];
    }

    public V put(K key, V value) {
        int index = key.hashCode() % default_length;// hascode值除map大小取余
        MyEntry<K, V> prevoius = entries[index];
        for (MyEntry<K, V> entry = entries[index]; entry != null; entry = entry.next) {
            if (entry.getKey().equals(key)) {
                V oldValue = (V) entry.getValue();
                entry.setValue(value);
                return oldValue;
            }
        }
        MyEntry<K, V> entry = new MyEntry<>(key, value);
        entry.next = prevoius;
        entries[index] = entry;
        return null;
    }

    public K get(K key){
        int index= key.hashCode()%default_length;
        for (MyEntry<K,V> entry= entries[index];entry!=null;entry=entry.next){
            if(entry.getKey().equals(key)){
                return (K)entry.getValue();
            }
        }
        return null;
    }

    private final class MyEntry<K, V> {
        private K key;
        private V value;
        private MyEntry next;

        public MyEntry() {
            super();
        }

        public MyEntry(K key, V value) {
            super();
            this.key = key;
            this.value = value;
        }

        public MyEntry(K key, V value, MyEntry next) {
            super();
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public K getKey() {
            return key;
        }

        public void setKey(K key) {
            this.key = key;
        }

        public V getValue() {
            return value;
        }

        public void setValue(V value) {
            this.value = value;
        }

        public MyEntry getNext() {
            return next;
        }

        public void setNext(MyEntry next) {
            this.next = next;
        }
    }
}
```

## 复杂版
```
public class MyHashMap {
    //默认初始化大小 16
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    //默认负载因子 0.75
    private static final float DEFAULT_LOAD_FACTOR = 0.75f;

    //临界值
    private int threshold;

    //元素个数
    private int size;

    //扩容次数
    private int resize;

    private MyEntry[] table;

    public MyHashMap() {
        table = new MyEntry[DEFAULT_INITIAL_CAPACITY];
        threshold = (int) (DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
        size = 0;
    }

    private int index(Object key) {
        //根据key的hashcode和entry长度取模计算key在entry中的位置
        return key.hashCode() % table.length;
    }

    public void put(Object key, Object value) {
        //key为null时需要特殊处理，为简化实现忽略null值
        if (key == null) return;
        int index = index(key);
        //遍历index位置的entry，若找到重复key则更新对应entry的值，然后返回
        MyEntry entry = table[index];
        while (entry != null) {
            if (entry.getKey().hashCode() == key.hashCode() && (entry.getKey() == key || entry.getKey().equals(key))) {
                entry.setValue(value);
                return;
            }
            entry = entry.getNext();
        }
        //若index位置没有entry或者未找到重复的key，则将新key添加到table的index位置
        add(index, key, value);
    }

    private void add(int index, Object key, Object value) {
        //将新的entry放到table的index位置第一个，若原来有值则以链表形式存放
        MyEntry entry = new MyEntry(key, value, table[index]);
        table[index] = entry;
        //判断size是否达到临界值，若已达到则进行扩容，将table的capacicy翻倍
        if (size++ >= threshold) {
            resize(table.length * 2);
        }
    }

    private void resize(int capacity) {
        if (capacity <= table.length) return;

        MyEntry[] newTable = new MyEntry[capacity];
        //遍历原table，将每个entry都重新计算hash放入newTable中
        for (int i = 0; i < table.length; i++) {
            MyEntry old = table[i];
            while (old!=null){
                MyEntry next = old.getNext();
                int index = index(old.getKey());
                old.setNext(newTable[index]);
                newTable[index] = old;
                old=next;
            }
        }
        //用newTable替table
        table = newTable;
        //修改临界值
        threshold = (int) (table.length * DEFAULT_LOAD_FACTOR);
        resize++;
    }

    public Object get(Object key){
        //这里简化处理，忽略null值
        if (key == null) return null;
        MyEntry entry= getEntry(key);
        return entry == null ? null : entry.getValue();
    }

    public MyEntry getEntry(Object key){
        MyEntry entry =table[index(key)];
        while (entry!=null){
            if (entry.getKey().hashCode()==key.hashCode()&&(entry.getKey()==key||entry.getKey().equals(key))){
                return entry;
            }
            entry = entry.getNext();
        }
        return entry;
    }
    public void remove(Object key) {
        if (key == null) return;
        int index = index(key);
        MyEntry pre = null;
        MyEntry entry = table[index];
        while (entry != null) {
            if (entry.getKey().hashCode() == key.hashCode() && (entry.getKey() == key || entry.getKey().equals(key))) {
                if (pre == null) table[index] = entry.getNext();
                else pre.setNext(entry.getNext());
                //如果成功找到并删除，修改size
                size--;
                return;
            }
            pre = entry;
            entry = entry.getNext();
        }
    }

    public boolean containsKey(Object key) {
        if (key == null) return false;
        return getEntry(key) != null;
    }

    public int size() {
        return this.size;
    }

    public void clear() {
        for (int i = 0; i < table.length; i++) {
            table[i] = null;
        }
        this.size = 0;
    }


    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(String.format("size:%s capacity:%s resize:%s\n\n", size, table.length, resize));
        for (MyEntry entry : table) {
            while (entry != null) {
                sb.append(entry.getKey() + ":" + entry.getValue() + "\n");
                entry = entry.getNext();
            }
        }
        return sb.toString();
    }
}

    final class MyEntry {
        private Object key;
        private Object value;
        private MyEntry next;

        public MyEntry(Object key, Object value, MyEntry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public Object getKey() {
            return key;
        }

        public void setKey(Object key) {
            this.key = key;
        }

        public Object getValue() {
            return value;
        }

        public void setValue(Object value) {
            this.value = value;
        }

        public MyEntry getNext() {
            return next;
        }

        public void setNext(MyEntry next) {
            this.next = next;
        }
    }
```