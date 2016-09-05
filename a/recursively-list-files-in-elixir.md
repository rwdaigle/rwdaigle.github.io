---
layout: post
title:  "Recursively List Files in Elixir"
date:   2016-09-05
---

Quick Elixir ditty to recursively list the files in a given directory:

```elixir
defmodule FileExt
  def ls_r(path) do
    cond do
      File.regular?(path) -> [path]
      File.dir?(path) ->
        File.ls!(path)
        |> Enum.map(&Path.expand(&1, path))
        |> Enum.map(&ls_r/1)
        |> Enum.reduce([], fn(files, acc) -> Enum.into(acc, files) end)
      true -> []
    end
  end
end
```

```elixir
FileExt.ls_r(".")     
["/Users/ryan/projects/rwdaigle.github.io/README.md",
 "/Users/ryan/projects/rwdaigle.github.io/LICENSE",
 "/Users/ryan/projects/rwdaigle.github.io/index.html",
 "/Users/ryan/projects/rwdaigle.github.io/Gemfile.lock",
 "..."]
```
