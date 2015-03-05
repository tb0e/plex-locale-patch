Plex Locale Patch
=================

[Plex] is a personal media server capable of playing remote content from the internet using [channels], that
are written in Python. While being great software it suffers from a few localization related issues:

  1. Some of the Plex clients tell their language the the Plex server incorrectly, resulting in having no translations.
  2. Translations retrieved from the channel's [Strings] may have incorrect encoding (even though the files are encoded
     correctly) which results in errors while preparing the XML output using the [lxml] library.  

These issues are probably going to be fixed in the future. Until that happens, this unofficial patch is available
to the channels authors. It should not break channels even after the issues are fixed.

What exactly is the problem with the Plex Framework
---------------------------------------------------

When the `Locale` object (`Framework.bundle/Contents/Resources/Versions/2/Python/Framework/api/localekit.py`) wants
to determine the current language, it ultimately requests the `locale` property of the `ExecutionContext` class instance
(`Framework.bundle/Contents/Resources/Versions/2/Python/Framework/code/context.py`), which actually returns a Request
header named `X-Plex-Language`:

    @property
    def locale(self):
        return self.get_header(Framework.constants.header.language)

It then looks for the translations in the [strings] JSON file with the same name.

The problem is that some clients (namely, Plex Home Theater) uses [ISO 639][iso-639] three-letter codes while the
[documentation][strings] mentions only two-letter language codes. Some other clients don't send that header at all,
but they may send the HTTP standard `Accept-Language` header that Plex ramework doesn't take into account.

Another problem is handling some unicode characters in the translated strings. Sometimes messages like this may be
observed in the log files:

    File "Framework.bundle/Contents/Resources/Versions/2/Python/Framework/modelling/objects.py", line 71, in _set_attribute
        el.set(convert_name(name), value)
    File "lxml.etree.pyx", line 699, in lxml.etree._Element.set (src/lxml/lxml.etree.c:34531)
    File "apihelpers.pxi", line 563, in lxml.etree._setAttributeValue (src/lxml/lxml.etree.c:15781)
    File "apihelpers.pxi", line 1366, in lxml.etree._utf8 (src/lxml/lxml.etree.c:22211)
    ValueError: All strings must be XML compatible: Unicode or ASCII, no NULL bytes or control characters 

The patch takes care of this issue as well.

How the patch works
-------------------

This patch detects the client language from `X-Plex-Language` and `Accept-Language` headers and converts the values
to the [ISO 639-1][iso-639] code format to make sure the correct translation file is used. The detected language is
injected back into the `X-Plex-Language` header from where its corrected value is handled by the Plex Framework natively. 

In addition, the translated strings are being properly decoded in order to avoid XML output errors as outlined above.

How to use
----------

The most simple way to add this patch to your channel is to install it as a git submodule:

  1. Open the `Contents/Code` directory of your channel.
  2. Execute this command:  
    `git submodule add https://bitbucket.org/czukowski/plex-locale-patch.git locale_patch`
  3. Open the `Code/__init__.py` file and add the following line:  
    `from locale_patch import L`
  4. Optionally use the following line instead of above:  
    `from locale_patch import L, SetAvailableLanguages`  
     Then call the `SetAvailableLanguages()` function with list of the available translations in the channel as a
     parameter from the channel's `Start()` function. This will help choose the correct language from the `Accept-Language`
     header. If this is not used, the locale with the greatest weight will be chosen from the header. 

That's it. From this point on, whenever the `L()` function is called from this file, the patched version is used. 

License
-------

This code is distributed under the MIT License.


[iso-639]: http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes#Partial_ISO_639_table
[channels]: https://plexapp.zendesk.com/hc/en-us/categories/200109616-Channels
[locale]: https://dev.plexapp.com/docs/api/localekit.html
[lxml]: http://lxml.de/
[plex]: https://plex.tv/
[strings]: https://dev.plexapp.com/docs/bundles/directories.html#the-strings-directory
