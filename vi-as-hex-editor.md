VI as HEX editor
=================

When running Mac OS X (or other *nix), you probably already have *vi* and *xxd* installed. Onwards!


Show as HEX in *vi*
-------------------

Put the following in *Command mode* <ESC>:

	:%!xxd

Voilà! *vi* filters the file (%) through the external (!) program *xxd*.

To return:

	:%!xxd -r


Map to keyboard
---------------

~/.vimrc:

	noremap <F8> :%!xxd<CR>
	noremap <F7> :%!xxd -r<CR>

Now you can switch between HEX mode using F8 and F7.

---

*Written by Prodhe*

