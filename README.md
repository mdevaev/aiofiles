# aiofiles: file support for asyncio

[![PyPI](https://img.shields.io/pypi/v/aiofiles.svg)](https://pypi.python.org/pypi/aiofiles)
[![Build](https://github.com/Tinche/aiofiles/workflows/CI/badge.svg)](https://github.com/Tinche/aiofiles/actions)
[![Coverage](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/Tinche/882f02e3df32136c847ba90d2688f06e/raw/covbadge.json)](https://github.com/Tinche/aiofiles/actions/workflows/main.yml)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/aiofiles.svg)](https://github.com/Tinche/aiofiles)
[![Black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

**aiofiles** is an Apache2 licensed library, written in Python, for handling local
disk files in asyncio applications.

Ordinary local file IO is blocking, and cannot easily and portably be made
asynchronous. This means doing file IO may interfere with asyncio applications,
which shouldn't block the executing thread. aiofiles helps with this by
introducing asynchronous versions of files that support delegating operations to
a separate thread pool.

```python
async with aiofiles.open('filename', mode='r') as f:
    contents = await f.read()
print(contents)
'My file contents'
```

Asynchronous iteration is also supported.

```python
async with aiofiles.open('filename') as f:
    async for line in f:
        ...
```

Asynchronous interface to tempfile module.

```python
async with aiofiles.tempfile.TemporaryFile('wb') as f:
    await f.write(b'Hello, World!')
```

## Features

- a file API very similar to Python's standard, blocking API
- support for buffered and unbuffered binary files, and buffered text files
- support for `async`/`await` ([PEP 492](https://peps.python.org/pep-0492/)) constructs
- async interface to tempfile module

## Installation

To install aiofiles, simply:

```bash
$ pip install aiofiles
```

## Usage

Files are opened using the `aiofiles.open()` coroutine, which in addition to
mirroring the builtin `open` accepts optional `loop` and `executor`
arguments. If `loop` is absent, the default loop will be used, as per the
set asyncio policy. If `executor` is not specified, the default event loop
executor will be used.

In case of success, an asynchronous file object is returned with an
API identical to an ordinary file, except the following methods are coroutines
and delegate to an executor:

- `close`
- `flush`
- `isatty`
- `read`
- `readall`
- `read1`
- `readinto`
- `readline`
- `readlines`
- `seek`
- `seekable`
- `tell`
- `truncate`
- `writable`
- `write`
- `writelines`

In case of failure, one of the usual exceptions will be raised.

`aiofiles.stdin`, `aiofiles.stdout`, `aiofiles.stderr`,
`aiofiles.stdin_bytes`, `aiofiles.stdout_bytes`, and
`aiofiles.stderr_bytes` provide async access to `sys.stdin`,
`sys.stdout`, `sys.stderr`, and their corresponding `.buffer` properties.

The `aiofiles.os` module contains executor-enabled coroutine versions of
several useful `os` functions that deal with files:

- `stat`
- `sendfile`
- `rename`
- `renames`
- `replace`
- `remove`
- `unlink`
- `mkdir`
- `makedirs`
- `rmdir`
- `removedirs`
- `link`
- `symlink`
- `readlink`
- `listdir`
- `scandir`
- `access`
- `path.exists`
- `path.isfile`
- `path.isdir`
- `path.islink`
- `path.getsize`
- `path.getatime`
- `path.getctime`
- `path.samefile`
- `path.sameopenfile`

### Tempfile

**aiofiles.tempfile** implements the following interfaces:

- TemporaryFile
- NamedTemporaryFile
- SpooledTemporaryFile
- TemporaryDirectory

Results return wrapped with a context manager allowing use with async with and async for.

```python
async with aiofiles.tempfile.NamedTemporaryFile('wb+') as f:
    await f.write(b'Line1\n Line2')
    await f.seek(0)
    async for line in f:
        print(line)

async with aiofiles.tempfile.TemporaryDirectory() as d:
    filename = os.path.join(d, "file.ext")
```

### Writing tests for aiofiles

Real file IO can be mocked by patching `aiofiles.threadpool.sync_open`
as desired. The return type also needs to be registered with the
`aiofiles.threadpool.wrap` dispatcher:

```python
aiofiles.threadpool.wrap.register(mock.MagicMock)(
    lambda *args, **kwargs: threadpool.AsyncBufferedIOBase(*args, **kwargs))

async def test_stuff():
    data = 'data'
    mock_file = mock.MagicMock()

    with mock.patch('aiofiles.threadpool.sync_open', return_value=mock_file) as mock_open:
        async with aiofiles.open('filename', 'w') as f:
            await f.write(data)

        mock_file.write.assert_called_once_with(data)
```

### History

#### 23.2.0 (UNRELEASED)

- aiofiles is now tested on Python 3.12 too.
  [#166](https://github.com/Tinche/aiofiles/issues/166) [#168](https://github.com/Tinche/aiofiles/pull/168)
- `aiofiles.tempfile.NamedTemporaryFile` now accepts a `delete_on_close` argument, just like the stdlib version.
- `aiofiles.tempfile.NamedTemporaryFile` no longer exposes a `delete` attribute, just like the stdlib version.

#### 23.1.0 (2023-02-09)

- Added `aiofiles.os.access`.
  [#146](https://github.com/Tinche/aiofiles/pull/146)
- Removed `aiofiles.tempfile.temptypes.AsyncSpooledTemporaryFile.softspace`.
  [#151](https://github.com/Tinche/aiofiles/pull/151)
- Added `aiofiles.stdin`, `aiofiles.stdin_bytes`, and other stdio streams.
  [#154](https://github.com/Tinche/aiofiles/pull/154)
- Transition to `asyncio.get_running_loop` (vs `asyncio.get_event_loop`) internally.

#### 22.1.0 (2022-09-04)

- Added `aiofiles.os.path.islink`.
  [#126](https://github.com/Tinche/aiofiles/pull/126)
- Added `aiofiles.os.readlink`.
  [#125](https://github.com/Tinche/aiofiles/pull/125)
- Added `aiofiles.os.symlink`.
  [#124](https://github.com/Tinche/aiofiles/pull/124)
- Added `aiofiles.os.unlink`.
  [#123](https://github.com/Tinche/aiofiles/pull/123)
- Added `aiofiles.os.link`.
  [#121](https://github.com/Tinche/aiofiles/pull/121)
- Added `aiofiles.os.renames`.
  [#120](https://github.com/Tinche/aiofiles/pull/120)
- Added `aiofiles.os.{listdir, scandir}`.
  [#143](https://github.com/Tinche/aiofiles/pull/143)
- Switched to CalVer.
- Dropped Python 3.6 support. If you require it, use version 0.8.0.
- aiofiles is now tested on Python 3.11.

#### 0.8.0 (2021-11-27)

- aiofiles is now tested on Python 3.10.
- Added `aiofiles.os.replace`.
  [#107](https://github.com/Tinche/aiofiles/pull/107)
- Added `aiofiles.os.{makedirs, removedirs}`.
- Added `aiofiles.os.path.{exists, isfile, isdir, getsize, getatime, getctime, samefile, sameopenfile}`.
  [#63](https://github.com/Tinche/aiofiles/pull/63)
- Added `suffix`, `prefix`, `dir` args to `aiofiles.tempfile.TemporaryDirectory`.
  [#116](https://github.com/Tinche/aiofiles/pull/116)

#### 0.7.0 (2021-05-17)

- Added the `aiofiles.tempfile` module for async temporary files.
  [#56](https://github.com/Tinche/aiofiles/pull/56)
- Switched to Poetry and GitHub actions.
- Dropped 3.5 support.

#### 0.6.0 (2020-10-27)

- `aiofiles` is now tested on ppc64le.
- Added `name` and `mode` properties to async file objects.
  [#82](https://github.com/Tinche/aiofiles/pull/82)
- Fixed a DeprecationWarning internally.
  [#75](https://github.com/Tinche/aiofiles/pull/75)
- Python 3.9 support and tests.

#### 0.5.0 (2020-04-12)

- Python 3.8 support. Code base modernization (using `async/await` instead of `asyncio.coroutine`/`yield from`).
- Added `aiofiles.os.remove`, `aiofiles.os.rename`, `aiofiles.os.mkdir`, `aiofiles.os.rmdir`.
  [#62](https://github.com/Tinche/aiofiles/pull/62)

#### 0.4.0 (2018-08-11)

- Python 3.7 support.
- Removed Python 3.3/3.4 support. If you use these versions, stick to aiofiles 0.3.x.

#### 0.3.2 (2017-09-23)

- The LICENSE is now included in the sdist.
  [#31](https://github.com/Tinche/aiofiles/pull/31)

#### 0.3.1 (2017-03-10)

- Introduced a changelog.
- `aiofiles.os.sendfile` will now work if the standard `os` module contains a `sendfile` function.

### Contributing

Contributions are very welcome. Tests can be run with `tox`, please ensure
the coverage at least stays the same before you submit a pull request.
