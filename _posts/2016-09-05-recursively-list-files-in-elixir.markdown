---
layout: post
title:  "Recursively List Files in Elixir"
date:   2016-09-05
author: ryan
tags:
  - elixir
category: blog
blog: true
---

Quick Elixir ditty to recursively list the files at a given path (relative or absolute):

```elixir
defmodule FileExt do
  def ls_r(path \\ ".") do
    cond do
      File.regular?(path) -> [path]
      File.dir?(path) ->
        File.ls!(path)
        |> Enum.map(&Path.join(path, &1))
        |> Enum.map(&ls_r/1)
        |> Enum.concat
      true -> []
    end
  end
end
```

```elixir
FileExt.ls_r("../exgen")
["../exgen/.git/COMMIT_EDITMSG", "../exgen/.git/config",
 "../exgen/.git/description", "../exgen/.git/HEAD",
 "../exgen/.git/hooks/applypatch-msg.sample",
 "..."]

 FileExt.ls_r("/Users/ryan/projects/exgen")
 ["/Users/ryan/projects/exgen/.git/COMMIT_EDITMSG",
  "/Users/ryan/projects/exgen/.git/config",
  "/Users/ryan/projects/exgen/.git/description",
  "/Users/ryan/projects/exgen/.git/HEAD",
  "/Users/ryan/projects/exgen/.git/hooks/applypatch-msg.sample",
  "..."]
```
