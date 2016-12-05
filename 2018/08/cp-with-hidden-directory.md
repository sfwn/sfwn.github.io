# cp -a with hidden directory

> Date:  2018-08-06
>
> Author: sfwn

reference: https://stackoverflow.com/a/4645159

> What you want is:
>
> cp -R t1/. t2/
> The dot at the end tells it to copy the contents of the current directory, > not the directory itself. This method also includes hidden files and folders.