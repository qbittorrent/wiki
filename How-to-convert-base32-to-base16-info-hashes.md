### Using info-hashes as download handles.

When interfacing qBittorrent through the WebUI api, there are cases when you add a download via a magnet URI and then you want to track its state (for instance, to perform certain file operation once it's finished). The problem here is that the /command/download method does not return a handle to the download for you to track it. On the other hand, webUI api methods refer to every download through its info-hash, so it's possible to use that string as a handle. As every magnet URI will contain, al least, the info-hash (as stated in [this](http://www.bittorrent.org/beps/bep_0009.html) document at BitTorrent.org) you can parse the hash string from the uri itself. These hashes are base16 encoded, for a total of 40 characters.  

### Special case: base32 encoded info-hashes.

For compatibility reasons, the same document states that clients should also support the 32 character base32 encoded info-hash. When adding such a magnet link, qBittorrent will automatically convert it to a base16 hash, and you end up with a mismatching handle (the original base32 encoded info-hash and the base16 one used by qBittorrent). In this situation it's necessary to previously re-encode the hash yourself. Basically you need to decode the base32 hash and then re-encode it to base16, this way you´ll have the same string as qBittorrent.

### Example:
Suppose you have the following magnet link:

    magnet:?xt=urn:btih:WRN7ZT6NKMA6SSXYKAFRUGDDIFJUNKI2

if it´s added to qBittorrent and then you issue a /query/torrents command, you´ll get a JSON object like this (fields other than "hash" are ommitted):
  
    [{...,"hash":"b45bfccfcd5301e94af8500b1a1863415346a91a",...},...]

which shows how qBittorrent converted our base32 hash to its base16 representation.

### How to implement the base conversion:

If you're into python, this base conversion can be accomplished as follows:
    
    >>> import base64
    >>> b32Hash = "WRN7ZT6NKMA6SSXYKAFRUGDDIFJUNKI2"
    >>> b16Hash = base64.b16encode(base64.b32decode(b32Hash))
    >>> b16Hash = b16Hash.lower()
    >>> print (b16Hash)

which will output: b'b45bfccfcd5301e94af8500b1a1863415346a91a'

In Delphi, you can make use of the excellent SZCodeBaseX library by Sasa Zeman (available for D4 to D2007, support seems to be dropped by now, but it's still available [here](http://www.torry.net/vcl/internet/coding/SZCodeBaseX.zip). This is a simple console application just to show how to perform the base conversion:

    program Project1;

    {$APPTYPE CONSOLE}

    uses
      SysUtils,
      SZCodeBaseX in '<path to the library>';

    var
      b32Hash: AnsiString;
      b16Hash: AnsiString;

    begin
      b32Hash:= 'WRN7ZT6NKMA6SSXYKAFRUGDDIFJUNKI2';
      b16Hash:= SZEncodeBase16(SZDecodeBase32(b32Hash));
      b16Hash:= LowerCase(b16Hash);
      Writeln(b16Hash);
    end.

If you compile and run Project1.exe, it´ll output the converted hash string.

Notice in both cases you need to perform a lowercase conversion of the resulting string.