---
layout: post
title: Belajar iOS Bagian 4
modified: 2017-06-3T20:15:28+07:00
categories:
description: Belajar iOS Bagian 4
tags: [Swift, iOS, Xcode, alamofire, RxSwift, ObjectMapper, Audio Player Manager, Marquee Label]
image:
  background: abstract-2.png
comments: true
share: true
date: 2017-05-22T20:15:28+07:00
---

Akhir nya kita bertemu lagi di artikel bagian 4 untuk development iOS :). Artikel ini adalah artikel bagian terakhir untuk pembahasan mengenai development iOS. Pada artikel ini, kita hanya akan membahas bagaimana cara memutar music yang berasal dari API spotify, ingat bahwa lagu yang akan diputar adalah hanya preview lagu saja, jadi lagu yang diputar tidak nya lengkap :).

Oke, silahkan buka project anda dengan xcode, nah disini kita perlu terlebih dahulu memisahkan antara view dan controller. Silahkan buat sebuah group dengan nama `controllers`, silahkan buka group `views`, disana terdapat file `TrackTableView` silahkan buka file tersebut, lalu copy isi source code nya lalu hapus file tersebut, silahkan buat file kembali dengan nama `TrackTableView` di dalam group controller dan paste kan isinya. Oke setelah selesai memindahkan nya, tahap selanjutnya silahkan buat sebuah file swift kembali di dalam group `controllers` dengan file `TrackPreviewViewController`.

Langkah selanjutnya kita akan terlebih dahulu membuat view, silahkan buat sebuah story board di dalam group `views` dengan nama `TrackPreview`, lalu untuk desainnya silahkan lihat video berikut.

<iframe width="560" height="315" src="https://www.youtube.com/embed/EiHnMxhal0w" frameborder="0" allowfullscreen></iframe>

Nah tahap desain udah selesai :D. Selanjutnya silahkan buka kembali file `TrackPreviewViewController` lalu isikan codingan seperti berikut.

{% highlight swift %}
//
//  TrackPreviewViewController.swift
//  Belajar-iOS
//
//  Created by rizki mufrizal on 5/11/17.
//  Copyright © 2017 rizki mufrizal. All rights reserved.
//

import UIKit
import AudioPlayerManager
import Kingfisher
import MarqueeLabel

class TrackPreviewViewController: UIViewController {

    @IBOutlet weak var buttonPlayPause: UIButton!
    @IBOutlet weak var labelArtist: MarqueeLabel!
    @IBOutlet weak var labelTrack: MarqueeLabel!
    @IBOutlet weak var imageAlbum: UIImageView!
    @IBOutlet weak var imageBackground: UIImageView!
    
    let track = UserDefaults.standard.object(forKey: "trackPreview") as! String
    let artistsName = UserDefaults.standard.object(forKey: "trackName") as! String
    let trackName = UserDefaults.standard.object(forKey: "artistName") as! String
    let image = UserDefaults.standard.object(forKey: "imageAlbum") as! String
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.title = "Belajar iOS"
        
        self.labelArtist.type = .continuous
        self.labelArtist.speed = .duration(15)
        self.labelArtist.animationCurve = .easeInOut
        
        self.labelTrack.type = .continuous
        self.labelTrack.speed = .duration(15)
        self.labelTrack.animationCurve = .easeInOut
        
        self.labelArtist.text = artistsName
        self.labelTrack.text = trackName
        
        let url = URL(string: image)
        self.imageBackground.kf.setImage(with: url)
        self.imageAlbum.kf.setImage(with: url)
        
        if AudioPlayerManager.shared.isPlaying() == true {
            AudioPlayerManager.shared.stop()
        }
        AudioPlayerManager.shared.play(urlString: track)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }

    @IBAction func playButtonAction(_ sender: Any) {
        if AudioPlayerManager.shared.isPlaying() == true {
            AudioPlayerManager.shared.pause()
            self.buttonPlayPause.setImage(UIImage(named: "play-icon"), for: .normal)
        } else {
            AudioPlayerManager.shared.play()
            self.buttonPlayPause.setImage(UIImage(named: "pause-icon"), for: .normal)
        }
    }
}
{% endhighlight %}

Untuk memutar music, penulis menggunakan library AudioPlayerManager :). Nah untuk mengambil data dari storyboard sebelumnnya, kita menggunakan fungsi `UserDefaults.standard`, fungsi ini sama seperti fungsi `LocalStorage` pada web browser. Untuk memutar musik, kita dapat menggunakan fungsi `AudioPlayerManager.shared.play`, baik itu musik di local mauapun melalui url, pada contoh kali ini kita menggunakan url. Untuk menganti gambar pada button, kita hanya perlu melakuka set image seperti perintah berikut.

{% highlight swift %}
self.buttonPlayPause.setImage(UIImage(named: "play-icon"), for: .normal)
{% endhighlight %}

Sehingga pada saat musik sedang berjalan, maka posisi button adalah pause, sedangkan pada saat musik berhenti maka posisi button adalah play.

Oke langkah selanjutnya, bagaimana cara untuk berpindah dari 1 storyboard ke storyboard yang lain ?. Ada beberapa cara yaitu bisa menggunakan `segue` atau menggunakan `pushViewController`, biasanya saya akan menggunakan `pushViewController`. Silahkan buka file `ViewController` lalu pada bagian extension `UITableViewDelegate` tambahkan kodingan berikut.

{% highlight swift %}
extension ViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        if indexPath.row == (self.rowData.count - 1) {
            if (self.nextPage ?? "").isEmpty == false {
                self.getDefaultTrack(track: track, offset: nextInt, limit: 20)
            }
        }
    }

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        if UserDefaults.standard.object(forKey: "trackPreview") != nil {
            UserDefaults.standard.removeObject(forKey: "trackPreview")
            UserDefaults.standard.synchronize()
        }
        if UserDefaults.standard.object(forKey: "trackName") != nil {
            UserDefaults.standard.removeObject(forKey: "trackName")
            UserDefaults.standard.synchronize()
        }
        if UserDefaults.standard.object(forKey: "artistName") != nil {
            UserDefaults.standard.removeObject(forKey: "artistName")
            UserDefaults.standard.synchronize()
        }
        if UserDefaults.standard.object(forKey: "imageAlbum") != nil {
            UserDefaults.standard.removeObject(forKey: "imageAlbum")
            UserDefaults.standard.synchronize()
        }

        UserDefaults.standard.set(self.rowData[indexPath.row].previewUrl, forKey: "trackPreview")
        UserDefaults.standard.set(self.rowData[indexPath.row].name, forKey: "trackName")
        UserDefaults.standard.set(ConvertString().fromArrayArtistToString(array: self.rowData[indexPath.row].artists!), forKey: "artistName")
        UserDefaults.standard.set(self.rowData[indexPath.row].album?.images?[0].url, forKey: "imageAlbum")
        UserDefaults.standard.synchronize()

        let storyboard = UIStoryboard(name: "TrackPreview", bundle: nil)
        let vc = storyboard.instantiateViewController(withIdentifier: "trackPreviewStoryBoard") as UIViewController
        self.navigationController?.pushViewController(vc, animated: true)
    }
}
{% endhighlight %}

fungsi dari function kedua diatas adalah jika user memilih salah satu row table, maka akan dijalankan aksi di dalam nya, di dalam function tersebut kita menyimpan data musik yang akan dikirim ke storyboard berikutnya, untuk berpindah ke storyboard kedua, kita menggunakan perintah berikut.

{% highlight swift %}
let storyboard = UIStoryboard(name: "TrackPreview", bundle: nil)
let vc = storyboard.instantiateViewController(withIdentifier: "trackPreviewStoryboard") as UIViewController
self.navigationController?.pushViewController(vc, animated: true)
{% endhighlight %}

Pertama kita deklarasikan terlebih dahulu variabel storyboard dengan value nama storyboard, tadinya kita membuat storyboard dengan nama `TrackPreview`, lalu buat sebuah variabel dengan menampung id dari storyboard tersebut, id ini bersifat unik walaupun berbeda storyboard, silahkan lihat video diatas pada saat deklarasikan storyboard id, storyboard id nya yaitu `trackPreviewStoryboard`. Selanjutnya untuk menampilkan storyboard berikutnya, kita menggunakan perintah `pushViewController` sehingga storyboard kedua akan berada diatas storyboard pertama sehingga akan saling tumpang tindih, konsep ini sebenarnya sama seperti konsep intent pada android :).

Berikut kodingan lengkap untuk `ViewController`

{% highlight swift %}
//
//  ViewController.swift
//  Belajar-iOS
//
//  Created by rizki mufrizal on 4/30/17.
//  Copyright © 2017 rizki mufrizal. All rights reserved.
//

import UIKit
import RxSwift
import Kingfisher
import SwiftOverlays

class ViewController: UIViewController {

    @IBOutlet weak var trackTable: UITableView!
    let searchBar = UISearchBar()

    var rowData: [Item] = []
    var nextPage: String? = nil
    var nextInt = 0
    var limitInt = 20
    var track = "ColdPlay"
    var isSearch = false

    private var disposeBag = DisposeBag()
    private var trackService = TrackService()

    override func viewDidLoad() {
        super.viewDidLoad()

        self.title = "Belajar iOS"
        self.getDefaultTrack(track: track, offset: nextInt, limit: limitInt)
        self.trackTable.delegate = self
        self.trackTable.dataSource = self

        self.searchBar.placeholder = "Cari"
        self.searchBar.delegate = self
        self.searchBar.tintColor = UIColor.black
        self.searchBar.sizeToFit()
        self.navigationItem.titleView = searchBar
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }

    func getDefaultTrack(track: String, offset: Int, limit: Int) {
        SwiftOverlays.showBlockingWaitOverlay()
        trackService
            .getTrackByQuery(parameters: ConvertString().toPlusString(text: track), offset: offset, limit: limit)
            .subscribe(
                onNext: { trackResponse in
                    if offset == 0 && trackResponse.tracks?.items?.count != 0 {
                        self.rowData.removeAll()
                    }
                    if trackResponse.tracks?.items?.count != 0 {
                        self.rowData.append(contentsOf: (trackResponse.tracks?.items)!)
                        if ((trackResponse.tracks?.next) ?? "").isEmpty == false {
                            self.nextPage = (trackResponse.tracks?.next)!
                        } else {
                            self.nextPage = nil
                        }
                        self.nextInt = (trackResponse.tracks?.offset)! + 20
                        self.trackTable.reloadData()
                    } else {
                        let alert = UIAlertController(title: "Info", message: "Maaf, Data Tidak ditemukan :(", preferredStyle: UIAlertControllerStyle.alert)
                        alert.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.default, handler: nil))
                        self.present(alert, animated: true, completion: nil)
                    }
                    SwiftOverlays.removeAllBlockingOverlays()
                },
                onError: { error in
                    SwiftOverlays.removeAllBlockingOverlays()
                }
            )
            .addDisposableTo(disposeBag)
    }

}

extension ViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        if indexPath.row == (self.rowData.count - 1) {
            if (self.nextPage ?? "").isEmpty == false {
                self.getDefaultTrack(track: track, offset: nextInt, limit: 20)
            }
        }
    }

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        if UserDefaults.standard.object(forKey: "trackPreview") != nil {
            UserDefaults.standard.removeObject(forKey: "trackPreview")
            UserDefaults.standard.synchronize()
        }
        if UserDefaults.standard.object(forKey: "trackName") != nil {
            UserDefaults.standard.removeObject(forKey: "trackName")
            UserDefaults.standard.synchronize()
        }
        if UserDefaults.standard.object(forKey: "artistName") != nil {
            UserDefaults.standard.removeObject(forKey: "artistName")
            UserDefaults.standard.synchronize()
        }
        if UserDefaults.standard.object(forKey: "imageAlbum") != nil {
            UserDefaults.standard.removeObject(forKey: "imageAlbum")
            UserDefaults.standard.synchronize()
        }

        UserDefaults.standard.set(self.rowData[indexPath.row].previewUrl, forKey: "trackPreview")
        UserDefaults.standard.set(self.rowData[indexPath.row].name, forKey: "trackName")
        UserDefaults.standard.set(ConvertString().fromArrayArtistToString(array: self.rowData[indexPath.row].artists!), forKey: "artistName")
        UserDefaults.standard.set(self.rowData[indexPath.row].album?.images?[0].url, forKey: "imageAlbum")
        UserDefaults.standard.synchronize()

        let storyboard = UIStoryboard(name: "TrackPreview", bundle: nil)
        let vc = storyboard.instantiateViewController(withIdentifier: "trackPreviewStoryboard") as UIViewController
        self.navigationController?.pushViewController(vc, animated: true)
    }
}

extension ViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.rowData.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = Bundle.main.loadNibNamed("TrackTableViewCell", owner: self, options: nil)?.first as! TrackTableViewCell

        cell.trackNameLabel.text = self.rowData[indexPath.row].name
        cell.artistNameLabel.text = ConvertString().fromArrayArtistToString(array: self.rowData[indexPath.row].artists!)
        cell.albumNameLabel.text = self.rowData[indexPath.row].album?.name

        let url = URL(string: (self.rowData[indexPath.row].album?.images?[0].url)!)
        cell.gambarAlbumImage.kf.indicatorType = .activity
        cell.gambarAlbumImage.kf.setImage(with: url)

        return cell
    }

    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 98
    }
}

extension ViewController: UISearchBarDelegate {
    func searchBarSearchButtonClicked(_ searchBar: UISearchBar) {
        self.track = self.searchBar.text!
        self.nextInt = 0
        self.getDefaultTrack(track: self.track, offset: self.nextInt, limit: self.limitInt)
        self.searchBar.resignFirstResponder()
        self.searchBar.setShowsCancelButton(false, animated: true)
        self.searchBar.resignFirstResponder()
    }

    func searchBarTextDidBeginEditing(_ searchBar: UISearchBar) {
        self.searchBar.setShowsCancelButton(true, animated: true)
    }

    func searchBarCancelButtonClicked(_ searchBar: UISearchBar) {
        self.searchBar.text = nil
        self.searchBar.setShowsCancelButton(false, animated: true)
        self.searchBar.resignFirstResponder()
    }
}
{% endhighlight %}

Sekarang silahkan jalankan, berikut adalah hasil output nya

![Screen Shot 2017-05-11 at 3.53.33 PM.png](../images/Screen Shot 2017-05-11 at 3.53.33 PM.png)

Wah tombol back nya kurang bagus nih :(, oke kita akan coba ganti tulisan nya menjadi `kembali` dengan warna hitam, silahkan buka file `TrackPreviewViewController` lalu tambahkan kodingan berikut di function `viewDidLoad`.

{% highlight swift %}
self.navigationController?.navigationBar.tintColor = UIColor.black
self.navigationController?.navigationBar.topItem?.title = "Kembali"
{% endhighlight %}

dan hasilnya seperti berikut :D.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Ef35KSBwykA" frameborder="0" allowfullscreen></iframe>

Akhirnya selesai juga artikel mengenai belajar iOS dari bagian 1 hingga 4 :). Untuk source code diatas dapat anda akses di [Belajar-iOS](https://github.com/RizkiMufrizal/Belajar-iOS/tree/bagian-4){:target="_blank"}. Sekian artikel mengenai Belajar iOS bagian 4, jika ada saran dan komentar silahkan isi dibawah dan terima kasih :)