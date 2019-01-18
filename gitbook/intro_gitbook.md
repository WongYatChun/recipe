# Step to build a gitbook

1. Go to the folder concerned
```
$ gitbook init
```

2. When you finish editting
```
$ gitbook serve
```

3. Build the book (but may not be necessary)
```
$ gitbook build
```

4. You may also specify the output port
```
$ gitbook serve --port 2333
```

5. Specify the output format
```
$ gitbook pdf ./ ./mybook.pdf
$ gitbook epub ./ ./mybook.epub
$ gitbook mobi ./ ./mybook.mobi
```