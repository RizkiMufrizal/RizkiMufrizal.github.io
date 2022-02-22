---
layout: post
title: Belajar iOS Bagian 3
modified: 2017-06-3T20:15:28+07:00
categories:
description: Belajar iOS Bagian 3
tags: [Swift, iOS, Xcode, alamofire, RxSwift, ObjectMapper, Search Bar]
image:
  background: abstract-2.png
comments: true
share: true
date: 2017-05-15T20:15:28+07:00
---

Sebelumnya kita telah membuat aplikasi untuk melihat list musik, akan tetapi kurang jika kita tidak menambahkan fitur search :). Oke, pada artikel ini kita akan membahas mengenai bagaimana cara membuat search :D. Mungkin artikel ini adalah artikel yang terpendek dikarenkan untuk membuat sebuah search bar di swift sangatlah gampang :). Oke kita langsung buka project iOS dengan xcode, project yang kita gunakan adalah project sebelum nya yang ada di artikel [sebelumnya](https://rizkimufrizal.github.io/belajar-ios-bagian-2/){:target="_blank"}.

Untuk membuat search bar silahkan buka file `ViewController`, kemudian deklarasikan sebuah variabel `searchBar` seperti berikut.

{% highlight swift %}
let searchBar = UISearchBar()
{% endhighlight %}

Setelah selesai, kita akan membuat search bar tersebut tepat di navigation menu, silahkan masukkan codingan berikut tepat di dalam function `viewDidLoad`

{% highlight swift %}
self.searchBar.placeholder = "Cari"
self.searchBar.delegate = self
self.searchBar.tintColor = UIColor.black
self.searchBar.sizeToFit()
self.navigationItem.titleView = searchBar
{% endhighlight %}

Nah karena diatas ada perintah `delegate` maka kita harus membuat 1 extension untuk menghadle action dari search bar, silahkan masukkan codingan extension berikut di bagian paling bawah.

{% highlight swift %}
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

fungsi function `searchBarSearchButtonClicked` berfungsi untuk menghandle action ketika user menekan tombol search, dan pada saat itu, kita akan menjalankan perintah `getDefaultTrack` untuk mengambil data musik. function kedua berfungsi memunculkan tombol cancel ketika kita melakukan editing pada search bar nya dan pada function terakhir ketika user menekan tombol cancel maka tombol cancel akan kembali seperti biasa. 

Berikut adalah hasil dari seluruh kodingannya.

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

Nah setelah selesai, silahkan jalankan aplikasi kalian, berikut adalah demo aplikasi nya :)

<iframe width="560" height="315" src="https://www.youtube.com/embed/khhhK9H13k8" frameborder="0" allowfullscreen></iframe>

Artikel selanjutnya akan membahas mengenai bagaimana cara untuk memutar lagu :D. Untuk source code diatas dapat anda akses di [Belajar-iOS](https://github.com/RizkiMufrizal/Belajar-iOS/tree/bagian-3){:target="_blank"}. Sekian artikel mengenai Belajar iOS bagian 3, jika ada saran dan komentar silahkan isi dibawah dan terima kasih :)