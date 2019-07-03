---
title: post
p: python/custom_module/file_getter
date: 2019-07-03 17:39:54
tags: ['file', 'python', 'custom module']
---

## 프로젝트 아래 특정하는 폴더 아래의 폴더, 파일 리스트를 검색하고, 완전한 path로 돌려받을 수 있는 모듈

```python
import os
import typing

ROOT_DIR: typing.Text
ROOT_DIR = os.path.abspath(os.path.dirname(os.path.dirname(__file__)))


def _get_all_children_path(folder_name: typing.Text) -> typing.Iterator[str]:
    abs_path: typing.Text
    abs_path = ROOT_DIR + "/" + folder_name
    return _map_path(_filter_avails(os.listdir(abs_path)), folder_name)


def _filter_avails(files: typing.List[str]) -> typing.Iterator[str]:
    condition: typing.Callable
    condition = lambda el: el[0] not in "._"
    return filter(condition, files)


def _filter_files(
    files: typing.Iterator[str], extension: typing.Text
) -> typing.Iterator[str]:
    condition: typing.Callable
    condition = lambda el: os.path.isfile(el) and extension in el
    return filter(condition, files)


def _filter_folders(files: typing.Iterator[str]) -> typing.Iterator[str]:
    condition: typing.Callable
    condition = lambda el: os.path.isdir(el)
    return filter(condition, files)


def _map_path(
    files: typing.Iterator[str], folder_name: typing.Text
) -> typing.Iterator[str]:
    condition: typing.Callable
    condition = (
        lambda el: f"{ROOT_DIR}/{el}"
        if folder_name is "."
        else f"{ROOT_DIR}/{folder_name}/{el}"
    )
    return map(condition, files)


def get_things(
    folder_name: typing.Text = ".",
    folder: bool = False,
    extension: typing.Optional[str] = None,
) -> typing.List[str]:
    """Get files and folder under single folder which you specified.

Keyword Arguments:
    folder_name {typing.Text} -- target folder name from ROOT (default: {"."})
    folder {bool} -- enable finding folders (default: {False})
    extension {typing.Optional[str]} -- enable finding files with extension string, if all insert as '' (default: {None})

Raises:
    FileNotFoundError: foldername not specified
    ValueError: specific file extension with folder not enable

Returns:
    typing.List[str] -- list out as full path

>>> helper_pys = get_things(folder_name="helper", extension=".py")
>>> any(['python_base/helper/path.py' in el for el in helper_pys])
True
>>> root_folders = get_things(folder=True, extension="")
>>> any(['python_base/helper' in el for el in root_folders])
True
    """
    avail_paths: typing.Iterator
    filterd: typing.Iterator[str]
    if not folder_name:
        raise FileNotFoundError("folder_name not specified")

    avail_paths = _get_all_children_path(folder_name)

    if (not folder) and (extension is not None):
        filterd = _filter_files(avail_paths, extension=extension)
    elif (extension is None) and folder:
        filterd = _filter_folders(avail_paths)
    elif (extension is "") and folder:
        filterd = avail_paths
    else:
        raise ValueError("extenstion with folder not available")

    return [_ for _ in filterd]


if __name__ == "__main__":
    import doctest
    doctest.testmod()
```