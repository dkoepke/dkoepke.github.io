---
title: "Object Sizes"
tags: [git, object, file size, repository, size, saving space, reducing size]
category: git
date: 2016-04-20
---

Files, even if deleted from the current tree, still consume space in your Git repository. You can look at the size of the packed files in history with:

```bash
git verify-pack -v .git/objects/pack/pack-*.idx | grep blob | sort -k4nr > object-sizes.txt
```

This will give you the SHA-1 and sizes (unpacked and packed) of all objects in the repository's history sorted by their packed size. I redirected it to a file `object-sizes.txt`. The hashes here are the ids of the file objects themselves not of commits. To map hashes to paths (with the caveat that these paths may not be in the current tree any longer), you need:

```bash
git rev-list --all --objects > object-hashes.txt
```

Then it's just a simple matter of grepping `object-hashes.txt`. These files can be quite large and take a while to produce, so it can be good to just cache them while you're investigating the repository size. Here's a simple script to get the top 20:

```bash
IFS=$'\n'

if [ ! -f object-sizes.txt ]; then
    git verify-pack -v .git/objects/pack/pack-*.idx | grep blob | sort -k4nr > object-sizes.txt
fi

if [ ! -f object-hashes.txt ]; then
    git rev-list --all --objects > object-hashes.txt
fi

largest=$(head -20 object-sizes.txt | cut -f 1,5,6 -d ' ')
output="size,packed,sha1,path"

for obj in $largest; do
    IFS=" " read obj_hash unpacked_size packed_size <<< "$obj"
    unpacked_size="$(($unpacked_size / 1024))kb"
    packed_size="$((packed_size / 1024))kb"
    path=$(grep $obj_hash object-hashes.txt | cut -f 2 -d ' ')
    output="${output}\n${unpacked_size},${packed_size},${obj_hash},${path}"
done

echo -e $output | column -t -s ', '
```
