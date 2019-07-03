## Custom module

- file_getter : 5개의 함수를 내부적으로 사용하여, 해당 폴더 아래 폴더, 혹은 특정 키워드의 파일을 리스트로  Fullpath로 출력해줍니다.
    ```python
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
    ```