---
title: "git sunucu servisleri"
excerpt: "Git sunucusu olarak kullanabileceginiz servisler ve ozelliklerinin karsilastirilmasi."
---

![[Kaynak](http://www.amarinfotech.com/gitlab-vs-github-vs-bitbucket.html)](/blog/assets/images/git-servisler/github-gitlab-bitbucket.jpeg)

*[Kaynak](http://www.amarinfotech.com/gitlab-vs-github-vs-bitbucket.html)*

Git’in ne olduğunu tartıştığımız [git nedir?]({% post_url 2017-07-13-git-nedir %}) adlı yazımızda bahsetmediğimiz, fakat bazı okuyucuların aklında canlanacak olan “lao biz git push yapıyoruz tamam da nereye yapıyoruz bunu?” sorusunu tartışmak istiyoruz bu yazıda.

Git’in takımlar arasında iş akışını hızlandırması ve tarafların sorunsuzca senkronize olabilmesini sağlamak için bir çeşit Git sunucusu gerekmekte. Bu sunucu, kullanıcıların yolladığı git repo’larını belirli bir düzen ve hiyerarşi içerisinde saklamalı, bu sayede de kodların kullanıcılar arasında paylaşılmasını sağlamalı. Bu noktada her şeyi çok basit hayal edebilirsiniz: uzak bir makinede boş bir git repo’su oluşturulacak, bir kullanıcıya bu repo’ya erişme izni verilecek, SSH kullanarak git push yapılacak, hiç bir problem kalmayacak. Bu şekilde bir setup yapmak için [şuradan ulaşılabilecek](https://www.linux.com/learn/how-run-your-own-git-server) İngilizce bir kaynak var, böyle bir sunucunun nasıl hazırlanacağını adım adım anlatmış, gayet temiz. Böyle bir Git sunucusu kodun paylaşılmasına izin veriyor vermesine fakat hem her repo için makineye SSH çekip manuel olarak repo’yu oluşturma ihtiyacı ortaya çıkıyor, hem de olur olmaz insanlara bu makineye erişme izni vermeniz gerekiyor, bu sürecin tümü de hızlı çalışmayı amaç haline getirmiş geliştiriciler için işkence haline geliyor.

Bu noktada çeşitli Git sunucu servisleri doğmuş işte. Aslında Amerika’yı yeniden keşfetmemişler, böyle bir ihtiyacın olduğu Git öncesi versiyon kontrol sistemlerinde de belli olduğu için varolan çözümlerin Git’e adapte edilmesi gerekmiş sadece. Bu süreç içerisinde çeşitli servisler gelmiş gitmiş, biz bu yazıda en popüler olan 3 tanesinden bahsedeceğiz. Bu servislerin avantajlarından ve dezavantajlarından bahsedeceğiz, bununla beraber hangi bütçe ve senaryolarda hangi servisin kullanılabileceği yönünde fikir vermeye çalışacağız.

## GitHub

![GitHub Logo]({{ '/assets/images/git-servisler/github-logo.png' | relative_url }})

Şüphesiz ki Git sunucu servislerinin en meşhurudur GitHub. [Sitelerinde yazdığına göre](https://github.com/about) **23 milyondan fazla kullanıcı ve 63 milyondan fazla proje varmış GitHub üzerinde.** En popüler olmasının altında sanıyorum bu alandaki ilklerden olması yatıyor, fakat bugün en popüler açık kaynak projelerin çoğu GitHub üzerinde tutuluyor, en önemli geliştirmeler, tartışmalar ve iş birlikleri GitHub üzerinde yaşanıyor, bu da platformu herkes için daha cazip bir hale getiriyor.

GitHub’ın en temel özelliklerinden biri açık kaynak geliştirme işini bir sosyal medyaya çevirmesi aslında. Sevdiğiniz geliştiricileri ve organizasyonları takip edebiliyor, projelerinde yaptıkları güncellemeleri ve geliştirmeleri okuyabiliyorsunuz. Aynı zamanda kullandığınız açık kaynak bir paket için siz de geliştirmeye katkıda bulunabiliyor veya geliştirenlere hata bildirimlerinde bulunarak yardımcı olabiliyorsunuz. Bugün bu gibi özellikler tüm popüler Git sunucu servislerinde mevcut, fakat bunları popüler hale getiren GitHub oldu.

### Maliyet

GitHub tamamen ücretsiz kullanılabilecek bir servis aslında. Public repo’lar için hiç bir ücret talep etmiyor GitHub, dolayısıyla sınırsız sayıda repo’nuz olabiliyor. Ücretlendirme ise private repo’lar oluşturmak istediğinizde karşınıza çıkıyor. Fiyatlar şu şekilde görünüyor:

![GitHub Pricing](/blog/assets/images/git-servisler/github-pricing.png)
*[Kaynak](https://github.com/pricing)*

Bununla beraber GitHub bir [Student Pack](https://education.github.com/pack) de sunuyor. Bu paket sayesinde eğer öğrenciyseniz çeşitli servislerin GitHub sayesinde size sunduğu özel servislerden faydalanabilirsiniz. Yine aynı paket ile yukarıdaki “Developer” pakedini de ücretsiz olarak kullanabiliyorsunuz.

**Guncelleme:** 2019 itibariyle GitHub private repo'lari de maksimum 3 contributor ile kullanilabilecek sekilde serbest birakmis bulunmakta.

**GitHub en popüler Git sunucu servisi şu an**, dolayısıyla güvenilirliği de yüksek. Eğer büyük projelerde çalışıyorsanız veya GitHub’ı sık sık kullanıyor ve “aaaaabii yeni bi sekmede başka servise mi geççeem şimdi yiaa” diyorsanız ücretli paketler işinizi görebilir. Ayriyetten “GitHub Enterprise” pakedi ile GitHub yazılımını satın alıp kendi sunucularınızda saklayabilirsiniz.

### Popüler Projeler

[GitHub Trending](https://github.com/trending) sayesinde GitHub üzerinde günlük, haftalık ve aylık olarak trend olan repo’ları görüntüleyebilir ve bunları dillere göre filtreleyebilirsiniz. Bunun dışında **GitHub en popüler açık kaynak projelere ev sahipliği yapıyor** demiştik. Bugün itibariyle [FreeCodeCamp](https://github.com/freeCodeCamp/freeCodeCamp), [Bootstrap](https://github.com/twbs/bootstrap), [React](https://github.com/facebook/react), [TensorFlow](https://github.com/tensorflow/tensorflow), [Vue.js](https://github.com/vuejs/vue), [Angular](https://github.com/angular/angular.js) gibi dev açık kaynak projeler GitHub üzerinde geliştiriliyorlar. Bunların dışında bahsetmemiz gereken belki de en önemli repository [Linus Torvalds’ın Linux çekirdek repo’su](https://github.com/torvalds/linux) olabilirdi, fakat GitHub üzerinde bu kaynak kodun sadece okunabilir bir versiyonu saklanmakta, geliştirilmeleri mail grubu üzerinden yapılmakta.

## Bitbucket

![Bitbucket Logo](/blog/assets/images/git-servisler/bitbucket-logo.png)

Piyasanın abisi [Atlassian](https://www.atlassian.com/)’ı bilen bilir. Bu reisleri **JIRA** ile bilirsiniz belki, bunun dışında **HipChat**, **Confluence** gibi ürünleri var. Son olarak da geçtiğimiz sene **Trello’yu satın aldı** babalar, böyle böyle içten fethediyorlar piyasayı. İşte bu **Bitbucket** da Atlassian’ın sunduğu, **Atlassian’ın diğer tüm ürünleri ile entegre çalışabilen bir Git sunucu servisi.** 2008 yılında kurulan Bitbucket, 2010'da Atlassian tarafından satın alınmış; ilk odağı Mercurial projeleri olan bu servis zaman içerisinde yüzünü Git projelerine dönmüş.

GitHub’ın aksine Bitbucket ücretsiz planında **sınırsız public ve private repo** sunan servislerden biri. Repo sayısı sınırlanmamış fakat bu plan içerisindeki takım maksimum 5 kişiden oluşabiliyor. Bu plan aslında küçük takımlar için ideal, bilhassa JIRA ve amcaoğullarını kullanıyorsanız iş akışınıza katkısı olacağı kesin. [Şurada](https://bitbucket.org/product/comparison/bitbucket-vs-github) görüldüğü kadarıyla Bitbucket kendini GitHub ile kıyaslamış, demiş ki “*GitHub’da code approval -takım arkadaşlarının kodlarının incelenmesi, bunun ardından beğenenlerin onaylaması- nanesi yok bizde var, bi de bizde continuous integration var*”. Code approval neyse de, continuous integration önemli mesele. Biliyorsunuz bu continuous integration mühim mesele. Temelde kod uzak sunucuya push’landığında testlerin koşturulması, eğer ilgili kod testleri başarıyla geçiyorsa da istenen branch’e uzak sunucu üzerinde merge’lenmesi pratiği olan continuous integration, [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines) ile tamamen Bitbucket üzerinden halledilebiliyormuş, merak eden detaylarını okuyabilir.

### Maliyet

Fiyatlandırması GitHub’a göre daha uygun olan Bitbucket planları ise şu şeklide:

![Bitbucket Pricing](/blog/assets/images/git-servisler/bitbucket-pricing.png)
*[Kaynak](https://bitbucket.org/product/pricing?tab=host-in-the-cloud)*

Bitbucket aynı zamanda güzel paralara **kendi sunucunuzda** kullanabileceğiniz versiyonunu da satıyor, ilgilisi kaynak linkinden detayları inceleyebilir.

Bitbucket da bu şekil, bilhassa Atlassian ürünü kullanan geliştirici takımlarını hedefleyen bir ürün. Kişisel projelerinizi saklamak için gayet kullanışlı bir servis, arayüzü falan da gayet temiz, denemeye değer. Popüler projeler hakkında bir fikrimiz yok, GitHub gibi açık kaynak yazılım merkezi değil Bitbucket, daha çok kendi kendine takılan kullanıcıyı hedefliyor.

## GitLab

![GitLab Logo](/blog/assets/images/git-servisler/gitlab-logo.png)

Geldik gönlümün efendisine: **GitLab**. GitLab 2011 yılında ilk adımları atılmış, başından beri **açık kaynak olarak geliştirilen** bir Git sunucu servisi. Reisler başlarda açık kaynak olarak geliştirmişler, sonra kullanıcı sayıları artmış vesaire. Bu süreçte bazı büyük organizasyonlar bu abilerden yeni özellikler talep etmişler, e bu abiler de “*taş mı yiyek*” demiş, GitLab Enterprise’ı çıkarmışlar. 2011'den bugüne kadar **proje halen açık kaynak olarak geliştiriliyor** ve bu açık kaynak olan versiyona **GitLab Community Edition** adı veriliyor. Velhasılıkelam, GitLab size tamamen ücretsiz bir alternatif sunuyor, temel özelliklerin hiç biri için para ödemeniz gerekmiyor.

### Maliyet

Aslında Community Edition’u indirip **kendi sunucunuzda** saklayabiliyorsunuz, fakat bu benim çok da umrumda olmayan bir özellik açıkçası. Bunun yerine, direk [gitlab.com](https://gitlab.com) üzerinden kayıt olduğunuzda **Enterprise Edition’u tamamen ücretsiz olarak kullanabiliyorsunuz**. Bu Enterprise Edition içerisinde de Free, Bronze, Silver ve Gold adında 4 farklı plan var, fakat şu an için Free ve Bronze planlar da Silver planın özelliklerini kullanabiliyormuş, gelecek dönemde kaldırılacak yazıyor sitelerinde. Fiyatlar ise şu şekilde:

![GitLab Pricing](/blog/assets/images/git-servisler/gitlab-pricing.png)
*[Kaynak](https://about.gitlab.com/gitlab-com/)*

GitLab de yine Bitbucket gibi continuous integration hizmeti sunuyor, direk sistem üzerinden continuous integration süreçlerini yönetebiliyorsunuz. Yine aynı şekilde sistem üzerinde kullanabildiğiniz Kanban board’da mevcut, tüm issue’larınızı çeşitli etiketlere göre board’lara ayırıp on numero iş takibi yapabiliyorsunuz. Bitbucket’taki approval falan alayı GitLab’de de var, üstüne işte bir kamyon fazlası var.

### Popüler Projeler

Bir kere [**GitLab Community Edition](https://gitlab.com/gitlab-org/gitlab-ce) tamamen Gitlab.com üzerinde saklanıyor**, en önemlisi bu sanıyorum. Bunun dışında [Gitter](https://gitlab.com/gitlab-org/gitter/), [TortoiseGit](https://gitlab.com/tortoisegit/tortoisegit/), [Inkscape](https://gitlab.com/inkscape/inkscape-web) gibi çeşitli büyük açık kaynak projeler de GitLab üzerinde saklanıyor. Tabii ki açık kaynak konusunda GitHub kadar popüler değil, fakat private repo’lar konusunda GitHub üzerine çok rahat tercih edilebilecek durumda.

## Sonuç

Yukarıda anlattık, dedik bu bunu yapar, şu şunu yapar. Her bir sistemin kendine göre artıları ve eksileri var, burada tercihi duruma göre yapmak gerekiyor. Eğer açık kaynak bir proje geliştiriyor ve bir komünite desteğine ihtiyaç duyuyorsanız **GitHub** en doğru çözüm olacaktır, eğer kısa zamanda güzel ilgi çekerseniz Trending sekmesi sayesinde çok iyi destek bulabilirsiniz projenize. Eğer bir takım olarak **JIRA**, **HipChat** gibi Atlassian ürünleri kullanıyorsanız ve projelerinizi özel repo’larda geliştirmek istiyorsanız **Bitbucket** çok temiz bir çözüm olacaktır sizin için, tüm platformlar ile tam entegrasyon içinde. Bunun dışında kalan irili ufaklı takımlar ve kişisel işlerin tümü için **GitLab** en doğru seçim gibi görünüyor. Ben bahsi geçen bu servislerin tümünü çeşitli amaçlar için kullanıyorum, kullanmaktan en keyif aldığım servis ise GitLab. Gerek proje yönetimi, gerek beleşe dünyaları sunmaları gibi detaylar bu hislerimde etken sanıyorum.

Eğer hatamız varsa affola, küçük araştırmalarla böyle bir derleme sunmak istedik. Yanlış gördüğünüz, düzeltilmesi gerektiğini düşündüğünüz bir nokta varsa yorumlardan veya *burak.karakan@gmail.com* adresinden bana ulaşabilirsiniz. Yazıyı inceleyip feedback verdiği için [Furkan Hatipoğlu](https://github.com/furkanhatipoglu) kankimi de öpüyorum burdan. İyi akşamlar cümleten, her nerede yaşıyor ve yaşatılıyor iseniz…

![Reha Muhtar](/blog/assets/images/git-servisler/reha.jpeg){: .center-image }

