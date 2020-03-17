---
title: Memento Pattern
layout: blog
category: [Design Patterns]
excerpt: In this blog, we will learn the design pattern named “Memento” with a real life use-case and how to use this pattern to solve that problem.
---

## Problem Statement

You have an editor in which you can save the content many times. You need an Undo feature in the editor with which you can undo the changes and get the previous content.

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var editor = new Editor();
        editor.setContent("First Content");
        editor.setContent("Second Content");
        editor.setContent("Third Content");
        editor.undo();
        Console.WriteLine(editor.getContent());
        Console.ReadKey();
    }
}
```

The output should be “Second Content”.

Put some thoughts on how you can achieve this using your programming language. Do not google and try to find out a solution. If you have achieved the solution/not able to figure out the solution, let’s now look at the solution approach.

## Solution Approach

![Memento UML](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/Memento.png)

## Explanation of the UML

- Editor is the class that stores the current version of the content. It can create a new content state when new data is provided as well as restore the state when undo is requested. In this case, Editor is called the Originator because editor is the one that takes the content & initiates the undo.
- To maintain the Single Responsibility Principle, we do not want to keep the previous states in the Editor class. Therefore, we bring an EditorState class that simply stores the state of the editor. EditorState class is useful if the editor needs to maintain the state of content & in future wants to also maintain state of something else in the editor.Because EditorState deals with only the state (which is what we want to do undo on), we call this as the Memento.
- The History class stores the list of EditorState objects and knows how to maintain the states and get the previous states. Thus, we call History class as the Caretaker.

## Implementation

```csharp
public class Editor
{
    private string content;

    public EditorState createState()
    {
        return new EditorState(content);
    }

    public void restore(EditorState state)
    {
        content = state.getContent();
    }

    public void setContent(string content)
    {
        this.content = content;
    }

    public string getContent()
    {
        return this.content;
    }
}

public class EditorState
{
    private readonly String content;

    public EditorState(String content)
    {
        this.content = content;
    }

    public String getContent()
    {
        return content;
    }
}

public class History
{
    private List<EditorState> states = new List<EditorState>();

    public void push(EditorState state)
    {
        states.Add(state);
    }

    public EditorState pop()
    {
        var lastState = states[states.Count - 1];
        states.Remove(lastState);
        return states[states.Count - 1];
    }
}

public class Program
{
    public static void Main(string[] args)
    {
        var editor = new Editor();
        var history = new History();
        editor.setContent("First Content");
        history.push(editor.createState());

        editor.setContent("Second Content");
        history.push(editor.createState());

        editor.setContent("Third Content");
        history.push(editor.createState());

        editor.restore(history.pop()); //this is the UNDO

        Console.WriteLine(editor.getContent());
        Console.ReadKey();
    }
}
```
