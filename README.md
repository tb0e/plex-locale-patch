Plex Locale Patch
=================

[Plex] is a personal media server capable of playing remote content from the internet using [channels], that
are written in Python. While being great software it suffers from a few localization related issues:

  1. Some of the Plex clients tell their language the the Plex server incorrectly, resulting in having no translations.
  2. Translations retrieved from the channel's [Strings] may have incorrect encoding (even though the files are encoded
     correctly) which results in errors while preparing the XML output using the [lxml] library.  

These issues are probably going to be fixed in the future. Until that happens, this unofficial patch is available
to the channels authors. It should not break channels even after the issues are fixed.

[channels]: https://plexapp.zendesk.com/hc/en-us/categories/200109616-Channels
[locale]: https://dev.plexapp.com/docs/api/localekit.html
[lxml]: http://lxml.de/
[plex]: https://plex.tv/
[strings]: https://dev.plexapp.com/docs/bundles/directories.html#the-strings-directory
