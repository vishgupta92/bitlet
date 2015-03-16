# Introduction #

This page is a sort of walkthrough describing how to use BitLet to download content from a given .torrent metafile.
It refers to the [Sample](http://code.google.com/p/bitlet/source/browse/src/main/java/org/bitlet/wetorrent/Sample.java) class contained within the project and it is intended to demonstrate how the library works.

# Sample Overview #

To download a torrent we need to read a torrent metafile first. This is done simply creating an instance of the class `Metafile` and passing to it an `InputStream` that will be used to read it:
```
// Parse the metafile
Metafile metafile = new Metafile(new BufferedInputStream(new FileInputStream(filename)));
```
The second step is to decide where the downloaded files are going to be written. We want to save the files on the disk, in the current working directory. The init method call will preallocate all the tree structure and the files on the disk.
```
// Create the torrent disk, this is the destination where the torrent file/s will be saved
TorrentDisk tdisk = new PlainFileSystemTorrentDisk(metafile, new File("."));
tdisk.init();
```
Before starting the download, we need to create and to start the `IncomingPeerListener`. This object will listen for new peer incoming connections on a specific port, and will dispatch them to the appropriate torrent. You can share this object between different `Torrent`s, so that you can have multiple torrent downloads listening for incoming tcp connections only on one port.
```
IncomingPeerListener peerListener = new IncomingPeerListener(PORT);
peerListener.start();
```
Now that we have the `Metafile`, the `TorrentDisk`, and the `IncomingPeerListener`, and we have specified a .torrent file, a path where to save the download, and a port to listen for incoming peer connections, we can start the download creating the `Torrent` object and calling the `startDownload` method:
```
Torrent torrent = new Torrent(metafile, tdisk, peerListener);
torrent.startDownload();
```
At this point the download has started and we have to call the tick function of the torrent regularly to update it.
```
while (!torrent.isCompleted()) {
    try {
        Thread.sleep(1000);
    } catch(InterruptedException ie) {
        break;
    }
    torrent.tick();
}
```
To stop the download, we can call the method `Torrent.interrupt`.

Please, remember to call the method `IncomingPeerListener.interrupt` too, to close the listening socket.