---
title: Linked List
layout: blog
category: [Fundamentals]
excerpt: Linked Lists are one of the most commonly used Linear Data Structures after arrays. They solve many problems of arrays and are used for building complex data structures. Unlike arrays, Linked Lists can grow and shrink automatically. A Linked List has a sequence of nodes. Every node has a value & address to next node...
comments: true
---

**Linked Lists** are one of the most commonly used **Linear Data Structures** after arrays. They solve many problems of arrays and are used for building complex data structures. Unlike arrays, Linked Lists can grow and shrink automatically.

- A Linked List has a **sequence of nodes**.
- Every **node has a value & address to next node** stored in it. So, we say that each node points to the next node. That is why, we refer to this data structure as LINKED LISTS.
- The **first node** is called **HEAD**.
- The **last node** is called **TAIL**.

![Linked List](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/linked-list.png)

## Runtime Complexity of Linked List Operations

- **Lookup by Value – O(n)**
  - Justification: The value that we are looking, in the worst case, may be at the end. So, we have to move from HEAD till TAIL to find the value. Thus, O(n).
- **Lookup by Index – O(n)**
  - Justification: Unlike arrays, the nodes are not sequentially allocated. As new items are added/deleted, the nodes are allocated/de-allocated. Thus, indexes are not in sequence. So, we again have to iterate from HEAD till TAIL to find out the node. Thus, O(n).
- **Insert Item at the End – O(1)**
  - Justification: We know that TAIL points to the last node which doesn’t have next node pointer in it. Just add a new node and have the TAIL item point to new node. Finally mark the new node as TAIL. Since, we didn’t have to iterate over the entire list. Thus, O(1).
- **Insert Item at the Beginning – O(1)**
  - Justification: We know that HEAD points to the first node. Just add a new node and point to the first node. Make the new node pointed as HEAD. Since, we didn’t have to iterate over the entire list. Thus, O(1).
- **Insert Item in the Middle – O(n)**
  - Justification: To find the position where new node has to be inserted requires an iteration. Thus, O(n).
- **Delete Item from the Beginning – O(1)**
  - Justification: Repoint the HEAD to the address of the next node stored in first node. Remove the address of the node being deleted. Since, we didn’t require iteration, Thus, O(1).
- **Delete Item from the End – O(n)**
  - Justification: Iterate from HEAD to TAIL and find out the last node attached to TAIL. During the iteration, save the reference of the second last node. Now, repoint TAIL to second last node and delete the address of next node. This makes the last node disconnected from linked list. Thus, O(n).
- **Delete Item from the Middle – O(n)**
  - Justification: Iterate from HEAD to the position of node which has to be deleted. Have the previous node point to next node of the node being deleted. Delete the next node reference from the node being deleted. Thus, O(n).

## Basic Program of how to work with Linked Lists

```csharp
LinkedList numbersList = new LinkedList();
numbersList.AddLast(10);
numbersList.AddLast(20);
numbersList.AddLast(30);
numbersList.AddFirst(5);
numbersList.RemoveLast();
numbersList.RemoveFirst();
Console.WriteLine(numbersList.IndexOf(20));
Console.WriteLine(numbersList.Contains(20));
Console.WriteLine(numbersList.Count);
foreach (var number in numbersList.ToArray())
{
    Console.WriteLine(number);
}
```

## Coding Exercise – Linked List

- Create a class “LinkedList” and another class “Node“
- The “LinkedList” class should have
  - **first** node
  - **last** node
  - **addFirst** method
  - **addLast** method
  - **removeFirst** method
  - **removeLast** method
  - **contains** method
  - **indexOf** method
  - **toArray** method
  - **count** property

> Spend some time thinking about the strategy and then start coding. Don’t copy-paste the code.

```csharp
public class LinkedList
{
    private Node First;
    private Node Last;
    private int size;

    public int Count
    {
        get { return size; }
    }
    public class Node
    {
        public int Value;
        public Node Next;

        public Node(int value)
        {
            Value = value;
        }
    }

    public void AddLast(int number)
    {
        var newNode = new Node(number);
        if (IsEmpty())
        {
            First = Last = newNode;
        }
        else
        {
            Last.Next = newNode;
            Last = Last.Next;
        }
        size++;
    }

    public void AddFirst(int number)
    {
        var newNode = new Node(number);
        if (IsEmpty())
        {
            First = Last = newNode;
        }
        else
        {
            newNode.Next = First;
            First = newNode;
        }
        size++;
    }

    private bool IsEmpty()
    {
        return First == null;
    }

    public void RemoveFirst()
    {
        if (IsEmpty())
            throw new System.Exception("No Such Element to Remove");
        if (First == Last)
        {
            First = Last = null;
        }
        else
        {
            var nodeToDelete = First;
            First = First.Next;
            nodeToDelete.Next = null;
        }
        size--;
    }

    public void RemoveLast()
    {
        if (IsEmpty())
            throw new System.Exception("No Such Element to Remove");
        if (First == Last)
        {
            First = Last = null;
        }
        else
        {
            var previous = GetPrevious(Last);
            Last = previous;
            Last.Next = null;
        }
        size--;
    }

    private Node GetPrevious(Node node)
    {
        var currentNode = First;
        while (currentNode != null)
        {
            if (currentNode.Next == node)
                return currentNode;
            else
                currentNode = currentNode.Next;
        }
        return null;
    }

    public bool Contains(int number)
    {
        return IndexOf(number) != -1;
    }

    public int IndexOf(int number)
    {
        int index = 0;
        var currentNode = First;
        while (currentNode != null)
        {
            if (currentNode.Value == number)
            {
                return index;
            }
            else
            {
                currentNode = currentNode.Next;
                index++;
            }
        }
        return -1;
    }

    public int[] ToArray()
    {
        int[] array = new int[size];
        var current = First;
        var index = 0;
        while (current != null)
        {
            array[index++] = current.Value;
            current = current.Next;
        }
        return array;
    }
}
```

## Difference between Arrays & Linked Lists

- Static Arrays have a fixed size
- Dynamic Arrays can grow by 50-100%
- Linked Lists don’t waste memory like Arrays
- Use Arrays if you know the number of items to store
- The time complexity of both data structures is different during different operations. Therefore, choose the data structure based on the problem that you are trying to solve.

## Types of Linked Lists

- **Singly Linked List** – A node in a singly linked list will only have a value & pointer to the next node.
- **Doubly Linked List** – A node in a doubly-linked list will have a value, pointer to next node & a pointer to the previous node. Hence, the doubly linked list reduces the O(n) complexity of RemoveLast operation in a singly linked list to O(1).
- **Circular Linked List** – Both the singly & doubly linked list can be circular. It means that the last node points to the first node.

## Coding Exercise: Reverse a Linked List

> Spend some time thinking about the strategy and then start coding. Don’t copy-paste the code.

```csharp
public void Reverse()
{
    if (IsEmpty())
        return;
    var prev = First;
    var current = First.Next;
    while (current != null)
    {
        var next = current.Next;
        current.Next = prev;
        prev = current;
        current = next;
    }

    Last = First;
    Last.Next = null;
    First = prev;
}
```

## Coding Exercise: Get the kth node from the end

> Spend some time thinking about the strategy and then start coding. Don’t copy-paste the code.

```csharp
public int GetKthFromTheEnd(int k)
{
    if (IsEmpty())
        throw new System.Exception("List is already empty");
    var n1 = First;
    var n2 = First;
    for (int i = 0; i < k - 1; i++)
    {
        n2 = n2.Next;
        if (n2 == null)
            throw new System.Exception("K is greater than the size of the list");
    }

    while (n2 != Last)
    {
        n1 = n1.Next;
        n2 = n2.Next;
    }

    return n1.Value;
}
```
