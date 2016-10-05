# Class 'fs/Path'

### Properties

* [$delimiter](#property-delimiter)
* [$separator](#property-separator)
* [$cwd](#property-cwd)

---

### Methods

* [@basename](#method-basename)
* [@dirname](#method-dirname)
* [@extname](#method-extname)
* [@format](#method-format)
* [@isAbsolute](#method-isAbsolute)
* [@join](#method-join)
* [@normalize](#method-normalize)
* [@parse](#method-parse)
* [@relative](#method-relative)
* [@resolve](#method-resolve)


--------------------------------------------------------------------------------

## $PROPERTY: delimiter

Provides the path delimiter.

---

#### Example

```ruby
${fs/Path:delimiter}
#--> ':'


$env:PATH
#--> '/usr/local/bin:/usr/bin:/bin'


$paths[^env:PATH.split[${fs/Path:delimiter};h]]
#--> |/usr/local/bin|/usr/bin|/bin|
```

--------------------------------------------------------------------------------

## $PROPERTY: separator

Provides the path segment separator.

---

#### Example

```ruby
${fs/Path:separator}
#--> '/'


$path[/assets/images/icons.png]
$parts[^path.split[${fs/Path:separator};h]]
#--> |assets|images|icons.png|
```

--------------------------------------------------------------------------------

## $PROPERTY: cwd

Contains the absolute path of the 'DOCUMENT_ROOT'.

---

#### Example

```ruby
${fs/Path:cwd}
#--> '/home/username/host/www'
```

--------------------------------------------------------------------------------
--------------------------------------------------------------------------------

## @METHOD: basename

The `^fs/Path:basename[]` methods returns the last portion of a `path`, similar to the Unix `basename` command.

---

### Syntax

`^fs/Path:basename[path[;ext]]`

### Params

* $path `<string>`
* $ext `<string>` An optional file extension.

### Example

```ruby
^fs/Path:basename[/foo/bar/baz/asdf/quux.html]
#--> 'quux.html'


^fs/Path:basename[/foo/bar/baz/asdf/quux.html;.html]
#--> 'quux'
```

--------------------------------------------------------------------------------

## @METHOD: dirname

The `^fs/Path:dirname[]` method returns the directory name of a `path`, similar to the Unix `dirname` command.

---

### Syntax

`^fs/Path:dirname[path]`

### Params

* $path `<string>`

### Example

```ruby
^fs/Path:dirname[/foo/bar/baz/asdf/quux]
#--> '/foo/bar/baz/asdf'
```

--------------------------------------------------------------------------------

## @METHOD: extname

The `^fs/Path:extname[]` method returns the extension of the `path`, from the last occurance of the `.` (period) character to end of string in the last portion of the `path`. If there is no `.` in the last portion of the `path`, or if the first character of the basename of `path` (see [`^fs/Path:basename[]`](#method-basename)) is `.`, then an empty string is returned.

---

### Syntax

`^fs/Path:extname[path]`

### Params

* $path `<string>`

### Example

```ruby
^fs/Path:extname[index.html]
#--> '.html'


^fs/Path:extname[index.coffee.md]
#--> '.md'


^fs/Path:extname[index.]
#--> '.'


^fs/Path:extname[index]
#--> ''


^fs/Path:extname[.index]
#--> ''
```

--------------------------------------------------------------------------------

## @METHOD: format

The `^fs/Path:format[]` method returns a path string from an `<hash>`. This is the opposite of [`^fs/Path:parse[]`](#method-parse).

---

### Syntax

`^fs/Path:format[hPath]`

### Params

* $hPath `<hash>`
	* $.dir `<string>`
	* $.root `<string>`
	* $.base `<string>`
	* $.name `<string>`
	* $.ext `<string>`

---

The following process is used when constructing the path string:
- `$result` is set to an empty string.
- If `$.dir[]` is specified, `$.dir[]` is appended to `$result` followed by the value of [`$fs/Path:separator`](#property-separator);
- Otherwise, if `$.root[]` is specified, `$.root[]` is appended to `$result`.
- If `$.base[]` is specified, `$.base[]` is appended to `$result`;
- Otherwise:
	- If `$.name[]` is specified, `$.name[]` is appended to `$result`
	- If `$.ext[]` is specified, `$.ext[]` is appended to `$result`.
- Return `$result`

### Example

```ruby
# If 'dir' and 'base' are provided:
# ${dir}${separator}${base}
# will be returned.
^fs/Path:format[
	$.dir[/home/user/dir]
	$.base[file.txt]
]
//--> '/home/user/dir/file.txt'


# 'root will be used if 'dir' is not specified.
# If only 'root' is provided or 'dir' is equal to 'root'
# then the separator will not be included.
^fs/Path:format[
	$.root[/]
	$.base[file.txt]
]
#--> '/file.txt'


# `name` + `ext` will be used if `base` is not specified.
^fs/Path:format[
	$.root[/]
	$.name[file]
	$.ext[.txt]
]
#--> '/file.txt'


# `base` will be returned if `dir` or `root` are not provided.
^fs/Path:format[
	$.base[file.txt]
]
#--> 'file.txt'
```

--------------------------------------------------------------------------------

## @METHOD: isAbsolute

The `^fs/Path:isAbsolute[]` method determines if `path` is an absolute path.

If the given `path` is a zero-length string, `false` will be returned.

---

### Syntax

`^fs/Path:isAbsolute[path]`

### Params

* path `<string>`

### Example

```ruby
^fs/Path:isAbsolute[/foo/bar]
#--> true


^fs/Path:isAbsolute[/baz/..]
#--> true


^fs/Path:isAbsolute[qux/]
#--> false


^fs/Path:isAbsolute[.]
#--> false
```

--------------------------------------------------------------------------------

## @METHOD: join

The `^fs/Path:join[]` method join all given `path` segments together using the [`$fs/Path:separator`](#property-separator) as a delimiter, then [@normalize](#method-normalize)s the resulting path.

Zero-length `path` segments are ignored. If the joined path string is a zero-length string then `'.'` will be returned, representing the current working directory.

---

### Syntax

`^fs/Path:join[path1;path2[;...]]`

### Params

* path[, ...] `<string>` A sequence of path segments.

### Example

```ruby
^fs/Path:join[/foo;bar;baz/asdf;quux;..]
#--> '/foo/bar/baz/asdf'


^fs/Path:join[foo; $.path[123] ;bar]
#--> throws 'invalid.argument' error.
```

--------------------------------------------------------------------------------

## @METHOD: normalize

The `^fs/Path:normalize[]` method normalizes the given `path`, resolving `'..'` and `'.'` segments.

When multiple, sequential path segment separation characters are found, they are replaced by a single instance of the path segment separator. Trailing separators are preserved.

If the path is a zero-length string, `'.'` is returned, representing the current working directory.

---

### Syntax

`^fs/Path:normalize[path]`

### Params

* path `<string>`

### Example

```ruby
^fs/Path:normalize[/foo/bar//baz/asdf/quux/..]
#--> '/foo/bar/baz/asdf'
```

--------------------------------------------------------------------------------

## @METHOD: parse

The `^fs/Path:parse[]` method returns an `<hash>` whose properties represent significant elements of the path.

The returned `<hash>` will have the following properties:

* $.root `<string>`
* $.dir `<string>`
* $.base `<string>`
* $.ext `<string>`
* $.name `<string>`

---

### Syntax

`^fs/Path:parse[path]`

### Params

* path `<string>`

### Example

```ruby
^fs/Path:parse[/home/user/dir/file.txt]
#-->
#  $.root[/]
#  $.dir[/home/user/dir]
#  $.base[file.txt]
#  $.ext[.txt]
#  $.name[file]
```

--------------------------------------------------------------------------------

## @METHOD: relative

The `^fs/Path:relative[]` method returns the relative path from `from` to `to`. If `from` and `to` each resolve to the same path (after calling [`^fs/Path:resolve[]`](#method-resolve) on each), a zero-length string is returned.

If a zero-length string is passed as `from` or `to`, the current working directory will be used instead of the zero-length strings.

---

### Syntax

`^fs/Path:relative[from;to]`

### Params

* from `<string>`
* to `<string>`

### Example

```ruby
^fs/Path:relative[/home/user/test/aaa;/home/user/impl/bbb]
#--> '../../impl/bbb'
```

--------------------------------------------------------------------------------

## @METHOD: resolve

The `^fs/Path:resolve[]` method resolves a sequence of paths or path segments into an absolute path.

The given sequence of paths is processed from right to left, with each subsequent `path` prepended until an absolute path is constructed. For instance, given the sequence of path segments: `/foo`, `/bar`, `baz`, calling  `^fs/Path:resolve[/foo;/bar;baz]` would return `/bar/baz`.

If after processing all given `path` segments an absolute path has not yet been generated, the current working directory is used.

The resulting path is normalized and trailing slashes are removed unless the path is resolved to the root directory.

Zero-length `path` segments are ignored.

If no `path` segments are passed, `^fs/Path:resolve[]` will return the absolute path of the current working directory.

---

### Syntax

`^fs/Path:resolve[[path[; ...]]`

### Params

* path[, ...] `<string>` A sequence of paths or path segments

### Example

```ruby
^fs/Path:resolve[/foo/bar;./baz]
#--> '/foo/bar/baz'


^fs/Path:resolve[/foo/bar;/tmp/file/]
#--> '/tmp/file'


# if the current working directory is '/home/myself/node'
^fs/Path:resolve[wwwroot;static_files/png/;../gif/image.gif]
#--> '/home/myself/node/wwwroot/static_files/gif/image.gif'
```

---