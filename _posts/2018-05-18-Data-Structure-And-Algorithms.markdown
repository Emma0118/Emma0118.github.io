---
layout: post
title: Data Structure and Algorithms in javascript: LinkedList
date: 2018-05-18 12:15:09
---

### 实现一个单向列表


```bash
let LinkedList = function() {

    let Node = function(element) {
        this.element = element;
        this.next = null;
    }
    let length = 0;
    let head = null;

    this.append = function(ele) {
        let element = new Node(ele);
        let current;
        if (head === null) {
            head = element
        } else {
            current = head;
            while (current.next) {
                current = current.next;
            }
            current.next = element;
        }
        length++;
    }
    this.getHead = function() {
        return head;
    }
    this.display = function() {
        var currNode = head;
        while (currNode) {
            document.write(currNode.element + '&nbsp;');
            currNode = currNode.next;
        }
    }

}

let list = new LinkedList();
list.append(3);
list.append(80);
list.display();

```