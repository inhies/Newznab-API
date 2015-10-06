Predefined Attributes
=====================

A set of known attributes for items in different categories has been defined.
Its possible that not all attributes are available at all times. Therefore a
client application should not rely on any particular attributes being in the
returned data but should take this list as an optional extra information.
However attributes marked with * are always available.

Additionally, not all attributes are applicable to all items. The category 
information can be used to check which attributes area available for which 
category items.

All attributes are defined using XML namespace syntax:
`xmlns:newznab="http://www.newznab.com/DTD/2010/feeds/attributes/"`

List of Attributes
------------------

Attribute       | Category               | Description                           | Example value
----------------|------------------------|---------------------------------------|------------------------------
size *          | ALL                    | Size in bytes                         | "252322"
category *      | ALL                    | Item's category                       | "5004"
guid            | ALL                    | Unique release guid                   | "6c6734da3e92a7b0e494e896b58081da"
files           | ALL                    | Number of files                       | "4"
poster          | ALL                    | NNTP Poster                           | "yenc@power-post"
group           | ALL                    | NNTP Group(s)                         | "a.b.group, a.b.teevee"
team            | ALL                    | Team doing the release                | "DiAMOND"
grabs           | ALL                    | Number of times item downloaded       | "1"
password        | ALL                    | Whether the archive is passworded     | "0" no, "1" rar pass, "2" contains inner archive
comments        | ALL                    | Number of comments                    | "2"
usenetdate      | ALL                    | Date posted to usenet                 | "Tue, 22 Jun 2010 06:54:22 +0100"
info            | ALL                    | Info (.nfo) file URL                  | "http://somesite/stuff/info.php?id=1234"
year            | ALL                    | Release year                          | "2009"
season          | TV                     | Numeric season                        | "1"
episode         | TV                     | Numeric episode within the season     | "1"
rageid          | TV                     | TVRage ID. (www.tvrage.com)           | "2322"
tvtitle         | TV                     | TVRage Show Title. (www.tvrage.com)   | "Duck and Cover"
tvairdate       | TV                     | TVRage Show Air date. (www.tvrage.com)| "Tue, 22 Jun 2010 06:54:22 +0100"
video           | TV, Movies             | Video codec                           | "x264"
audio           | TV, Movies, Audio      | Audio codec                           | "AC3 2.0 @ 384 kbs"
resolution      | TV, Movies             | Video resolution                      | "1280x716 1.78:1"
framerate       | TV, Movies             | Video fps                             | "23.976 fps"
language        | TV, Movies, Audio      | Natural languages                     | "English"
subs            | TV, Movies             | Subtitles                             | "English, Spanish"
imdb            | Movies                 | IMDb ID  (www.imdb.com)               | "0104409"
imdbscore       | Movies                 | IMDb score                            | "5/10"
imdbtitle       | Movies                 | IMDb score                            | "Bobs Movie"
imdbtagline     | Movies                 | IMDb tagline                          | "Bobs new adventure"
imdbplot        | Movies                 | IMDb plot                             | "All about the movie"
imdbyear        | Movies                 | IMDb year                             | "1971"
imdbdirector    | Movies                 | IMDb director                         | "Bob Smith"
imdbactors      | Movies                 | IMDb actors                           | "Bob Smith, Kate Smith"
genre           | TV, Movies             | Genre                                 | "Horror, Comedy"
artist          | Music                  | Artist name                           | "Bob Smith"
album           | Music                  | Album name                            | "Groovy Tunes"
publisher       | Music, Book            | Publisher name                        | "Epic Music"
tracks          | Music                  | Track listing                         | "track one|track two|track three"
coverurl        | TV, Movies, Music, Book| URL to cover image                    | "http://servername.com/covers/music/12345.jpg"
backdropcoverurl| TV, Movies, Music      | URL to backdrop image                 | "http://servername.com/covers/movies/12345-backdrop.jpg"
review          | Movies, Music, Book    | Critics review                        | "This media is great"
booktitle       | Book                   | Book title                            | "Weather and Folk Lore of Peterborough and District"
publishdate     | Book                   | Date book published                   | "Tue, 22 Jun 2010 06:54:22 +0100"
author          | Book                   | Book author                           | "Charles Dack"
pages           | Book                   | Number of pages                       | "123"

Attribute Example
-----------------

Example attribute declarations within `<item>` element:

    <newznab:attr name="category" value="2000" /> 
    <newznab:attr name="category" value="2030" /> 
    <newznab:attr name="size"     value="4294967295" /> 
