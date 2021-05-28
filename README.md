# Multicast-Calisma-Notlarim-Cisco

# MULTICAST
Geleneksel IP haberleşmesinde ağ kullanıcıları iletim için farklı metotlar kullanır. Bunlar:
- Unicast (one-to-one)
- Broadcast (one-to-all)
- Multicast (one-to-many)

Multicast iletişim, ağ bant genişliği kullanımını optimize eden ve sistem kaynaklarını koruyan bir teknolojidir. Katman 2 ağlarda çalışması için Internet Group Management Protocol (IGMP) ‘e ve Katman 3 ağlarda çalışması için Protocol Independent Multicast (PIM) ‘e dayanır.

### MULTICAST ADRESLEME
Internet Assigned Number Authority (IANA), Multicast adresleme için D sınıfı IP adresi alanından 224.0.0.0/4 ‘ü atamıştır. 224.0.0.0 ile 239.255.255.255 arasında değişen adresler, multicast trafiği içindir. Tüm bu aralığın ilk 4 biti 1110 ile başlaması gerekir.
Aşağıda bilinen rezerve edilmiş multicast adresleri listelenmiştir.
224.0.0.0 ->Temel adres
224.0.0.1 -> Ağdaki tüm kullanıcılar
224.0.0.2 -> Ağdaki tüm routerlar
224.0.0.5 -> Tüm OSPF çalıştıran routerlar
224.0.0.6 -> Tüm OSPF çalıştıran DR routerlar
224.0.0.9 -> Tüm RIPv2 çalıştıran routerlar
224.0.0.10 -> Tüm EIGRP çalıştıran routerlar
224.0.0.13 -> Tüm PIM routerlar
224.0.0.18 -> VRRP
224.0.0.22 -> IGMPv3
224.0.0.102 -> HSRPv2 ve GLBP
224.0.1.1 -> NTP
224.0.1.39 -> Cisco-RP-Announce (Auto-RP)

### LAYER2 MULTICAST ADRESLER
Bir LAN segmentindeki NIC’ler (Ethernet kartı) yalnızca kendi lokalindeki MAC adreslerine veya broadcast MAC adreslerine yönelik paketler alabilir. Multicast için farklı bir yöntem oluşturulması gerekir.

MAC adresleri, bir LAN segmentindeki NIC’i benzersiz şekilde tanımlamak için kullanılan 48 bit uzunluğunda değerlerdir. Multicast trafik için, MAC adresinin ilk 24 biti her zaman 01:00:5E ile başlar. MAC adresinin ilk byte’ının least significant biti multicast için her zaman 1 olmalıdır, buna da unicast/multicast biti denir. Ayrıca MAC adresinin 25.biti multicast adresleme için daima 0 olmalıdır. Geri kalan 23 bit ise multicast IP adresinin son 23 bitinden kopyalanır. Böylece Layer2 Multicast Adres oluşturulmuş olur.

Örneğin;
Multicast IP adresi 239.255.1.1 olan bir multicast grubu için multicast MAC adresi 01:00:5E:7F:01:01 ‘ dir.
IP adresinin binary formatta yazalım ->
11101111.11111111.00000001.00000001
Son 23 bit : 1111111.00000001.00000001
Multicast MAC adresi 01:00:5E ile başlayıp, 25 biti 0 olup, geri kalan bitler ise IP adresinin son 23 bitini alarak oluşturulmalıdır.
Multicast MAC adresi binary formatı ->
00000001-00000000-01011110-0 1111111-00000001-00000001

###  INTERNET GROUP MANAGEMENT PROTOCOL (IGMP)
IGMP, alıcıların multicast gruplarına katılmak ve bu gruplardan trafik almaya başlamak için kullandıkları protokoldür. Bir alıcı, bir kaynaktan multicast trafiği almak istediğinde routera bir “IGMP join” gönderir. Eğer routerda IGMP etkin değilse, istek yok sayılır. IGMP’nin 3 versiyonu bulunmaktadır. IGMPv1, eski ve az kullanılan versiyondur.
##### IGMPv2
IGMPv2’de mesaj formatındaki TTL (time-to-live) değeri 1 olarak verilir, sebebi başka routerlara bu mesajın iletilmemesini sağlamaktır. Çünkü TTL değeri 255’e kadar verilebilir ve her hop geçtiğinde bir azaltılır, sıfır olduğunda ise gönderim durur.

IGMPv2’de mesaj formatında bulunan alanlar şunlardır:

- Type: Bu alan, yönlendiriciler ve alıcılar tarafından kullanılan beş farklı tipte IGMP mesajını açıklar. Bunlar:
     _Version 2 membership report (type value 0x16)_, genellikle IGMP join olarak da adlandırılan bir ileti türüdür. Alıcılar tarafından bir multicast grubuna katılmak veya lokal bir yönlendiricinin membership query mesajına yanıt vermek için kullanılır.

     _Version 1 membership report (type value 0x12)_, IGMPv1 ile uyumluluk için, alıcılar tarafından kullanılır.
     
     _Version 2 leave group (type value 0x17)_, alıcılar tarafından katıldıkları bir grup için multicast trafiği almayı durdurmak istediklerini belirtmek için kullanılır.
     
    _General membership query (type value 0x11)_, alt ağda herhangi bir alıcı olup olmadığını görmek için grup adresine 224.0.0.1 düzenli olarak gönderilir. Grup adresi alanını 0.0.0.0 olarak ayarlar.
    
    _Group specific query (type value 0x11)_, alıcının ayrılmayı talep ettiği grup adresine bir group leave mesajına yanıt olarak gönderilir. Grup adresi, IP paketinin hedef IP adresi ve grup adresi alanıdır.
    
    _Max response time_: Bu alan yalnızca genel ve gruba özgü membership query iletilerinde ayarlanır (type value 0x11); saniyenin onda biri birimlerinde yanıt veren bir rapor göndermeden önce izin verilen maksimum süreyi belirtir. Diğer tüm mesajlarda, gönderen tarafından 0x00 olarak ayarlanır ve alıcılar tarafından yok sayılır.
    
    _Checksum_: TCP / IP tarafından kullanılan standart checksum algoritmasıdır.
    
    _Group address_: Bu alan, general query mesajlarında 0.0.0.0 olarak ayarlanmıştır ve gruba özel mesajlarda grup adresi olarak ayarlanmıştır. Membership report mesajları, bu alanda rapor edilen grubun adresini taşır; Group leave mesajları, bu alanda bırakılan grubun adresini taşır.
    
    
Bir alıcı multicast trafiği almak istediğinde, katılmak istediği grup için lokal yönlendiriciye (örneğin, 239.1.1.1) genellikle IGMP join olarak adlandırılan unsolicited membership report gönderir. Lokal yönlendirici daha sonra bu isteği bir PIM join mesajı kullanarak kaynağa doğru gönderir. Lokal yönlendirici, multicast trafiğini almaya başladığında, onu isteyen alıcının bulunduğu alt ağa iletir.

##### IGMPv3
IGMPv2'de, bir alıcı multicast grubuna katılmak için bir membership report gönderdiğinde, hangi kaynaktan multicast trafiği almak istediğini belirtmez. IGMPv3, alıcılara multicast trafiğini kabul etmek istedikleri kaynağı seçme olanağı veren multicast kaynak filtrelemesi için destek ekleyen bir IGMPv2 uzantısıdır. IGMPv3, IGMPv1 ve IGMPv2 ile birlikte var olacak şekilde tasarlanmıştır.

IGMPv3, tüm IGMPv2’nin IGMP mesaj türlerini destekler ve IGMPv2 ile uyumludur. İkisi arasındaki fark, IGMPv3'ün IGMP membership query’e yeni alanlar eklemesi ve kaynak filtrelemeyi desteklemek için Version 3 membership report adı verilen yeni bir IGMP mesaj türü sunmasıdır. IGMPv3 kullanılmasının amacı, kaynak filtrelemesini sağlamasıdır. (Source Specific Multicast SSM)

### IGMP SNOOPING
Trafiği optimize etmek ve flooding’i önlemek için switchlerin yalnızca ilgili alıcılara trafik gönderme yöntemine ihtiyacı vardır. Unicast trafikte, Cisco switchler MAC adres kaynağını inceleyerek MAC adreslerini ve hangi bağlantı noktalarına ait olduklarını öğrenirler ve bu bilgileri MAC adres tablosunda saklarlar. Bu tabloda olmayan bir hedef MAC adresi için bir frame alınırsa bu “unknown frame” olarak kabul edilip frame’in alındığı arayüz hariç aynı VLAN içindeki tüm bağlantı noktalarına gönderirler. Buna da kısaca “flooding” denir. İlgili MAC adresindeki cihaz dışındaki tüm cihazlar paketi atarlar.

Multicast trafiği olduğunda, multicast MAC adresleri hiçbir zaman kaynak MAC adresi olarak kullanılmaz. Switchler multicast MAC adreslerini unknown frame olarak ele alır ve flooding olur. Bu durum LAN segmentinde bant genişliği kullanımını boşa harcar.

Cisco switchler, bir LAN segmentinde multicast flooding'i azaltmak için iki yöntem kullanır. Bunlar:
- IGMP Snooping
- Statik MAC adres girişleri

IGMP Snooping, alıcılar tarafından gönderilen IGMP join’leri inceleyerek IGMP join arayüzlerini içeren bir tablo tutarak çalışır. Switch, multicast grubuna yönelik bir multicast frame’i aldığında paketi yalnızca o multicast grubu için iletir.

### PROTOCOL INDEPENDENT MULTICAST
Alıcılar, multicast grubuna katılmak için IGMP kullanır; bu, grubun kaynağı, alıcının bağlı olduğu routera bağlanması yeterlidir. Yönlendiricilerin diğer yönlendiricilerden multicast akışlarını bulabilmesi ve talep edebilmesi, ve multicast trafiğini ağ boyunca yönlendirmesi için bir multicast yönlendirme protokolü gereklidir. Birden çok multicast yönlendirme protokolü mevcuttur, ancak Cisco yalnızca Protocol Independent Multicast (PIM)’i tam olarak destekler.

PIM, ağ segmentleri arasında multicast trafiğini yönlendiren bir protokolüdür. PIM, kaynak ve alıcılar arasındaki yolu belirlemek için unicast yönlendirme protokollerinden herhangi birini kullanabilir.

##### PIM DISTRIBUTION TREES

Multicast routerları, alıcılara ulaşmak için multicast trafiğinin ağ üzerinden izlediği yolu tanımlayan bir “Distribution Trees (Dağıtım Ağacı) ” oluşturur. Bu Distribution Trees, shortest path trees (en kısa yol ağacı) (SPTs) ve shared trees (paylaşılan ağaç) olmak üzere ikiye ayrılır.
##### _Source Trees_
Source trees, kaynağın ağacın kökü olduğu ve dalların ağ üzerinden alıcılara kadar bir dağıtım ağacı oluşturduğu multicast dağıtım ağacıdır. Bu ağaç inşa edildiğinde, kaynaktan ağacın yapraklarına kadar ağ üzerinden en kısa yol kullanır; bu nedenle, en kısa yol ağacı (SPT) olarak da adlandırılır.

SPT'nin iletim durumu, "S comma G" olarak telaffuz edilir ve gösterimi (S, G) şeklindedir; burada S, multicast akışının kaynağıdır ve G, multicast grup adresidir.

##### _Shared Trees_
Shared trees, kökünün rendezvous point (buluşma noktası) (RP) olarak belirlenen bir router olduğu multicast dağıtım ağacıdır. Bu nedenle, bu türe RP trees (RPTs) de denir. Multicast trafiği, kaynak adresinden bağımsız olarak, paketlerin adreslendiği grup adresine göre shared trees’ten aşağıya iletilir. Shared trees’de iletim durumu "star comma G" olarak telaffuz edilir ve (*, G) gösterimiyle belirtilir.

#### PIM TERMINOLOGY
- Reverse Path Forwarding (RPF) interface: RPF interface, source trees’de kaynak IP adresine, shared trees’de RP IP adresine giderken en düşük maliyete sahip arabirimdir. Birden fazla arabirim aynı maliyete sahipse, en yüksek IP adresine sahip arabirim, eşitliği bozan arabirim olarak seçilir.
 
- RPF nedighbor: RPF arayüzündeki PIM komşusudur.

- Incoming Interface (IIF): Kaynaktan gelen multicast trafiğini kabul edebilen arabirim türü, RPF arabirimiyle aynıdır.

- Outgoing Interface (OIF): Multicast trafiğini ağaç boyunca iletmek için kullanılan herhangi bir arabirim, downstream arabirimi olarak da bilinir.

- Outgoing interface list (OIL): Multicast trafiğini aynı gruba ileten bir OIF grubudur.

- Last-Hop Router (LHR): Alıcılara doğrudan bağlı olan ve leaf router olarak da bilinen bir yönlendiricidir. PIM join’lerini RP'ye veya kaynağa doğru göndermekten sorumludur.

- First-Hop Router (FHR): Kök yönlendirici (root router) olarak da bilinen, doğrudan kaynağa bağlı bir yönlendiricidir. RP'ye register (kayıt) mesajları göndermekle sorumludur.

- Multicast Routing Information Base (MRIB): Unicast yönlendirme tablosu ve PIM'den türetilen multicast yol tablosu (mroute) olarak da bilinen bir topoloji tablosudur. MRIB, her multicast yolu için kaynak S, grup G, gelen arabirimler (IIF), giden arabirimler (OIF'ler) ve RPF komşu bilgilerini içerir.

- Multicast Forwarding Information Base (MFIB): Daha hızlı yönlendirme ve multicast yönlendirme bilgilerini programlamak için MRIB'yi kullanan bir yönlendirme tablosudur.

- Multicast state: Bir yönlendirici tarafından multicast trafiğini iletmek için kullanılan durumlardır. Multicast durumu, mroute tablosunda (S, G, IIF, OIF vb.) bulunan girişlerden oluşur.

5 adet PIM modu bulunmaktadır. Bunlar:
- PIM Dense Mode (PIM-DM)
- PIM Sparse Mode (PIM-SM)
- PIM Sparse Dense Mode
- PIM Source Specific Multicast (PIM-SSM)
- PIM Bidirectional Mode (Bidir-PIM)

#### PIM CONTROL MESSAGE TYPES
PIM control mesajları IP protokol numarası 103’ü kullanır. Aşağıdaki tabloda PIM control mesaj tipleri gösterilmiştir.
```sh
 TYPE  MESSAGE TYPE       DESTINATION (Hedef)                PIM PROTOCOL
 0    	Hello 	          224.0.0.13(tüm PIM routerlar)    PIM-SM, PIM-DM,Bidir-PIM ve SSM 
 1  	  Register 	        RP adresi (unicast) 	           PIM-SM 
 2 	    Register stop 	  First-hop router (unicast) 	     PIM-SM 
 3 	    Join/prune 	      224.0.0.13 (tüm PIM routerlar)   PIM-SM, Bidir-PIM ve SSM 
 4 	    Bootstrap 	      224.0.0.13 (tüm PIM routerlar)   PIM-SM ve Bidir-PIM 
 5 	    Assert 	          224.0.0.13 (tüm PIM routerlar)   PIM-SM, PIM-DM, ve Bidir-PIM 
 8 	    Candidate RP adv 	Bootstrap router (BSR) adresi    PIM-SM ve Bidir-PIM 
 9 	    State fresh 	    224.0.0.13 (tüm PIM routerlar)   PIM-DM 
 10 	  DF election 	    224.0.0.13 (tüm PIM routerlar)   Bidir-PIM 
```

PIM hello mesajları, tüm PIM router adreslerine komşu PIM routerları hakkında bilgi edinmek için her etkin arabirimden her 30 saniyede bir gönderilir. Tüm PIM routerları, her bir PIM komşusundan alınan hello bilgilerini kaydetmelidir.

##### PIM DENSE MODE (PIM-DM)

PIM-DM, multicast trafiğini ağın her köşesine taşımak için bir “push” modeli kullanır. PIM-DM, ağda bulunan her alt ağda aktif alıcıların bulunduğu belirli dağıtımlarda verimli olacaktır.

PIM-DM, başlangıçta ağdaki multicast trafiğini flood eder. Downstream komşusu olmayan routerlar, istenmeyen trafiği geri çeker. Bu işlem her 3 dakikada bir tekrarlanır. Routerlar, flood ve brune mekanizması aracılığıyla veri akışlarını alarak durum bilgilerini toplar. Bu veri akışları, kaynak ve grup bilgilerini içerir, böylece downstream routerlar kendi multicast yönlendirme tablolarını oluşturabilirler. PIM-DM yalnızca source trees, yani (S, G) girişlerini destekler ve shared trees oluşturmak için kullanılamaz.

##### PIM SPARSE MODE (PIM-SM)
PIM-SM, multicast trafiği sağlamak için bir “pull” modeli kullanır. Yalnızca, verileri açıkça talep eden etkin alıcılara sahip ağ segmentleri trafiği alacaktır. PIM-SM, veri paketlerini shared trees’te ileterek etkin kaynaklar hakkındaki bilgileri dağıtır. PIM-SM shared trees kullandığından, bir rendezvous point (RP) kullanılması gerektirir. RP, ağda yönetimsel olarak yapılandırılmalıdır. PIM-SM'de rendezvous point (RP) olarak, bir veya daha fazla router seçmek zorunludur. Bir RP, her routerda statik olarak yapılandırılabilir veya dinamik bir mekanizma yoluyla öğrenilebilir.

##### _Static RP_
RP adresini multicast etki alanındaki her routerda yapılandırarak, RP'yi bir multicast grubu için statik olarak yapılandırmak mümkündür. Statik RP'lerin yapılandırılması nispeten basittir ve her routerda bir veya iki konfigürasyon satırı ile gerçekleştirilebilir. Ağda tanımlanmış birden fazla RP yoksa veya RP'ler çok sık değişmiyorsa, RP'leri tanımlamak için en basit yöntem bu olabilir. Ağ küçükse de iyi bir seçenek olabilir.
Ancak, statik yapılandırma, büyük ve karmaşık bir ağda yönetim yükünü artırabilir. Her routerın aynı RP adresine sahip olması gerekir. Bu, RP adresinin değiştirilmesinin her router için yeniden yapılandırılma gerektirdiği anlamına gelir. Farklı gruplar için birkaç RP etkinse, hangi RP'nin hangi multicast grubunu işlediği hakkındaki bilgiler tüm routerlar tarafından bilinmelidir. Bu bilgilerin eksiksiz olduğundan emin olmak için birden çok yapılandırma komutu gerekebilir.
##### _Auto-RP_
Auto-RP, bir PIM ağında, gruptan RP'ye erişimleri otomatikleştiren Cisco'ya özel bir mekanizmadır. Otomatik RP aşağıdaki faydalara sahiptir:
- Farklı grup aralıklarına hizmet vermek için bir ağ içinde birden çok RP kullanmak kolaydır.
- Farklı RP'ler arasında yük bölünmesine izin verir.
- Grup katılımcılarının konumlarına göre RP yerleştirmeyi basitleştirir.
- Bağlantı sorunlarına neden olabilecek tutarsız manuel statik RP yapılandırmalarını önler.
- Birden çok RP, farklı grup aralıklarına hizmet etmek veya birbirlerinin yedekleri olarak hizmet etmek için kullanılabilir.
- Auto-RP mekanizması, iki temel bileşenden oluşur. Bunlar; candidate RP (C-RP) ve mapping agent (MA).

##### _Candidate RP_
Bir C-RP, RP announcement (duyuru) mesajları aracılığıyla RP olma isteğini ilan eder. Bu mesajlar, varsayılan olarak 60 saniyede bir multicast grubuna 224.0.1.39 (Cisco RPAnnounce) gönderilir. Birden fazla C-RP varsa, en yüksek IP adresine sahip C-RP tercih edilir.
##### _RP Mapping Agent_
RP MA'lar, RP duyurularını almak için 224.0.1.39 grubuna katılır. Duyuruların içerdiği bilgileri bekletme süreleriyle birlikte group-to-RP mapping önbelleğinde saklarlar. Birden fazla RP aynı grup aralığının reklamını yaparsa, en yüksek IP adresine sahip C-RP seçilir.

RP MA'ları, RP eşlemelerinin bir başka multicast grubu adresi olan 224.0.1.40'a (Cisco-RP-Discovery) reklamını yapar. Bu mesajlar varsayılan olarak her 60 saniyede bir veya değişiklikler algılandığında duyurulur. MA duyuruları, seçilmiş RP'leri ve group-to-RP eşleştirmeleri içerir. Tüm PIM etkin yönlendiriciler 224.0.1.40'a katılır ve RP eşlemelerini özel önbelleklerinde depolar.

Arıza durumunda yedeklilik sağlamak için aynı ağda birden fazla RP MA yapılandırılabilir. Aralarında seçim mekanizması yoktur ve birbirlerinden bağımsız hareket ederler. Hepsi, PIM etki alanındaki tüm yönlendiricilere aynı group-to-RP mapping bilgilerinin reklamını yapar.

#### PIM BOOTSTRAP ROUTER
RFC 5059'da açıklanan bootstrap router (önyükleme yönlendirici) (BSR) mekanizması, hataya dayanıklı, otomatikleştirilmiş RP keşif ve dağıtım mekanizması sağlayan bir mekanizmadır. PIM, her grup prefixi için RP küme bilgilerini keşfetmek ve bir PIM etki alanındaki tüm yönlendiricilere duyurmak için BSR'yi kullanır. Bu, Auto-RP ile gerçekleştirilen aynı işlevdir, ancak BSR, PIM Sürüm 2 spesifikasyonunun bir parçasıdır.

Hatadan kaçınmak için, bir PIM alanında birden çok candidate BSR (C-BSR) dağıtılabilir. Tüm C-BSR'ler, BSR önceliklerini içeren PIM BSR mesajlarını tüm arayüzlere göndererek BSR seçim sürecine katılırlar. En yüksek önceliğe sahip C-BSR, BSR olarak seçilir ve PIM alanındaki tüm PIM yönlendiricilerine BSR mesajları gönderir. BSR öncelikleri eşitse veya BSR önceliği yapılandırılmamışsa, en yüksek IP adresine sahip C-BSR, BSR olarak seçilir.
##### CANDIDATE RP
Bir candidate RP (C-RP) olarak yapılandırılan bir yönlendirici, o anda aktif olan BSR'nin IP adresini içeren BSR mesajlarını alır. BSR'nin IP adresini bildiği için, C-RP candidate RP advertisement (C-RPAdv) mesajlarını doğrudan tek noktaya yayınlayabilir. Bir C-RPAdv mesajı, bir grup adresi çiftleri listesi taşır. Bu, bir C-RP'nin RP olmak istediği grup aralıklarını belirlemesini sağlar.

Etkin BSR, gelen tüm C-RP reklamlarını group-to-RP mapping önbelleğinde depolar. BSR daha sonra tüm C-RP'lerin listesini BSR mesajlarında group-to-RP mapping önbelleğinden varsayılan olarak ağdaki tüm PIM yönlendiricilerine her 60 saniyede bir gönderir. Yönlendiriciler bu BSR mesajlarının kopyalarını aldıkça, yerel group-to-RP mapping önbelleklerindeki bilgileri günceller ve bu, ağdaki tüm C-RP'lerin IP adreslerini tam olarak görmelerini sağlar.

MA’nın bir grup aralığı için etkin RP'yi seçtiği ve seçim sonuçlarını ağa duyurduğu Auto-RP'den farklı olarak, BSR bir grup için etkin RP'yi seçmez. Bunun yerine, bu görevi ağdaki her bir yönlendiriciye bırakır.

Ağdaki her yönlendirici, belirli bir grup aralığı için halihazırda aktif olan RP'yi seçmek için bir karma algoritma kullanır. Her yönlendirici aynı C-RP listesine karşı aynı algoritmayı çalıştırdığından, hepsi belirli bir grup aralığı için aynı RP'yi seçecektir. Daha düşük öncelik değerine sahip C-RP'ler tercih edilir. Öncelikler aynıysa, en yüksek IP adresine sahip C-RP, belirli grup aralığı için RP olarak seçilir.
