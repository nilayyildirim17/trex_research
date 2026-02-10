# ARAŞTIRMA & RAPORLAMA ÖDEVİ (.NET)

.NET Backend Geliştirme – Temel Bilgi ve Kavramlar

---

## 1. Modern Yazılım Geliştirme Pratikleri

### Git nedir? GitHub nedir?

Git bir versiyon kontrol sistemidir. Projenin her zaman son haline ulaşmamızı ve güncel tutmamızı sağlayan bir araçtır. Projenin bir çok kişi ile aynı anda yapılabilmesini sağlar.

Github ise projelerimizin saklandığı uzak sunucudur. Github'a projelerinizi ekleyebilir aynı zamanda istediğiniz public olan farklı projelere Github üzerinden erişerek projeyi bilgisayarınıza indirebilirsiniz. Hatta istediğinizde bu projeler üzerinde değişiklikler yaparak Pull Request gönderebilirsiniz.

### Temel Git komutları

**git init**: Bir Repository oluşturur ve biz bu Repositort'i Github'a gönderebiliriz.
```bash
git init
```

**git clone**: Uzak sunucudaki bir projeyi bilgisayarımıza indirmek istediğimizde kullanırız.
```bash
git clone <proje linki>
```

**git add**: Bu komut ile yaptığımız değişiklikler Git'e eklenir.
```bash
git add deneme.txt
```

**git commit**: Bu komut ile yapılan değişiklikler local Repository'e kaydedilir.
```bash
git commit -m "commit mesajı"
```

**git push**: Localde yapılan değişiklikleri uzak sunucuya (Github vs.) gönderme işlemidir.
```bash
git push
```

**git pull**: Uzak sunucudaki değişiklikleri veya herhangi bir projeyi localimize çekmek için kullanılır.
```bash
git pull origin master
```

**git branch**: Komutu ile yeni branch oluştururuz.
```bash
git branch development
```

**git merge**: İki veya daha fazla geliştirme geçmişini birleştirmektir.
```
          A---B---C topic
         /
    D---E---F---G master
```

### Merge conflict nedir, nasıl çözülür?

Merge conflict, Git iki branch'i birleştirirken aynı dosyanın aynı satırlarında farklı değişiklikler olduğunu gördüğünde ortaya çıkar.

Birleştirme çakışmasını çözmenin en doğrudan yolu, çakışan dosyayı düzenlemektir. Dosyayı merge.txt favori düzenleyicinizde açın. Örneğimizde, tüm çakışan ayırıcıları kaldıralım. Değiştirilmiş merge.txt içerik şu şekilde görünmelidir:

```
this is some content to mess with
content to append
totally different content to merge later
```

Dosya düzenlendikten sonra, git add merge.txt birleştirilmiş yeni içeriği hazırlamak için kullanın. Birleştirmeyi tamamlamak için şu komutu çalıştırarak yeni bir commit oluşturun:

```bash
git commit -m "merged and resolved the conflict in merge.txt"
```

Git, çakışmanın çözüldüğünü görecek ve birleştirmeyi tamamlamak için yeni bir birleştirme taahhüdü oluşturacaktır.

### CI/CD nedir?

CI/CD, Continuous Integration (Sürekli Entegrasyon) ve Continuous Deployment (Sürekli Dağıtım) anlamına gelir.

**CI (Continuous Integration)**: Kodunuzu her commit attığınızda otomatik olarak test edilir, build alınır

**CD (Continuous Deployment)**: Test edilen kod otomatik olarak production ortamına deploy edilir

#### Azure DevOps

**Artıları:**
- Microsoft ekosistemiyle mükemmel entegrasyon
- Güçlü YAML pipeline sistemi
- Ücretsiz private repository'ler
- Enterprise-grade güvenlik

**Eksileri:**
- UI biraz karmaşık olabilir
- Learning curve var

**Ne zaman kullanmalı**: .NET projeleri, enterprise ortamlar, Microsoft Azure kullanıyorsanız

**Pipeline Örneği:**

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop
  paths:
    exclude:
      - README.md
      - docs/*

pool:
  vmImage: 'ubuntu-latest'

variables:
  nodeVersion: '18.x'
  npmCache: $(Pipeline.Workspace)/.npm

stages:
  - stage: Build
    displayName: 'Build ve Test'
    jobs:
      - job: BuildJob
        displayName: 'Build ve Test İşlemleri'
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(nodeVersion)
            displayName: 'Node.js $(nodeVersion) Kurulumu'

          - task: Cache@2
            inputs:
              key: 'npm | "$(Agent.OS)" | package-lock.json'
              restoreKeys: |
                npm | "$(Agent.OS)"
              path: $(npmCache)
            displayName: 'NPM Cache'

          - script: |
              npm ci --cache $(npmCache) --prefer-offline
            displayName: 'NPM Dependencies Yükleme'

          - script: |
              npm run lint
            displayName: 'Linting Kontrolleri'

          - script: |
              npm run test:coverage
            displayName: 'Unit Testler ve Coverage'

          - task: PublishTestResults@2
            condition: succeededOrFailed()
            inputs:
              testRunner: JUnit
              testResultsFiles: 'coverage/junit.xml'
            displayName: 'Test Sonuçlarını Yayınla'

          - task: PublishCodeCoverageResults@1
            inputs:
              codeCoverageTool: Cobertura
              summaryFileLocation: 'coverage/cobertura-coverage.xml'
            displayName: 'Coverage Raporunu Yayınla'

          - script: |
              npm run build
            displayName: 'TypeScript Build'

          - task: PublishBuildArtifacts@1
            inputs:
              PathtoPublish: 'dist'
              ArtifactName: 'app-build'
            displayName: 'Build Artifacts Yayınla'
```

#### GitHub Actions

**Artıları:**
- GitHub ile native entegrasyon
- Büyük marketplace ekosistemi
- Kolay kullanım
- Open source projeler için ücretsiz

**Eksileri:**
- Private repo'larda dakika sınırı
- Bazen yavaş olabiliyor

**Ne zaman kullanmalı**: Open source projeler, GitHub kullanıyorsanız, basit workflow'lar

**Pipeline Örneği:**

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Node.js Setup
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install Dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Test
        run: npm run test:coverage
      
      - name: Build
        run: npm run build
      
      - name: Upload Coverage
        uses: codecov/codecov-action@v3

  deploy-staging:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Staging
        run: echo "Staging'e deploy ediliyor..."

  deploy-production:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Production
        run: echo "Production'a deploy ediliyor..."
```

### Software Development Life Cycle (SDLC)

Yazılım geliştirme sürecinin aşamaları (Planlama, analiz, geliştirme, test, dağıtım, bakım)

#### 1. Planlama
Proje kapsamını, hedeflerini ve gereksinimleri belirlenir. (İlk proje planı)

Proje planlama aşaması, bir yazılım geliştirme projesinin hedeflerini ve kapsamını belirler.

#### 2. Analiz
Proje gereksinimleri hakkında veri toplanır ve gözden geçirilir. (Tamamen ayrıntılı gereksinim belgeleri)

Analiz aşamasında, geliştirme ekibi projenin gereksinimleri hakkında bilgi toplar ve analiz eder. Analiz, ekibin planlama aşamasında başladıkları işi ilerletmesini ve üst düzey bir fikirden pratik bir uygulama planına geçmesini sağlar.

#### 3. Tasarım
Proje mimarisini tanımlar. Yazılım tasarım belgesi (SDD)

Tasarım aşaması, projenin mimarisini tanımlamayı içerir. Temel adımlar, yazılımın navigasyonunu, kullanıcı arayüzlerini, veritabanı tasarımını ve daha fazlasını özetlemeyi içerir.

#### 4. Kodlama
Başlangıç kodunu yazar. Fonksiyonel yazılım prototipidir.

Kodlama aşaması veya geliştirme aşaması, ekibin önceki aşamalarda oluşturulan SDD, SRS ve diğer yönergelere dayanarak kodu yazmaya ve yazılımı oluşturmaya başlamasıdır.

#### 5. Test
Kodu gözden geçirir ve hatalar ortadan kaldırılır. Rafine, optimize edilmiş yazılımdır.

Test aşaması, geliştirme ekibi işlevsel bir yazılım parçası oluşturduktan sonra başlar. Bu aşamada ekip, hataları ortadan kaldırmak ve nihai ürünü geliştirmek için fırsatlar arar.

#### 6. Dağıtım
Kodu üretim ortamına dağıtır. Son kullanıcılar için mevcut yazılımdır.

Dağıtım aşamasında, kullanıcıların erişebileceği üretim ortamına ince ayarlı yazılım dağıtılır.

#### 7. Bakım
Sürekli düzeltmeler ve iyileştirmeler yapılır. Güncellenmiş ve optimize edilmiş koddur.

Bakım aşaması, yazılım ekiplerinin yazılımın sürekli çalışmasını sağlamaya yardımcı olmak için yaptığı dağıtım sonrası çalışmaları gerektirir: güncellemeleri, öngörülemeyen değişiklikler yapmak, yeni kullanım durumlarını ele almak ve kullanıcıların bulduğu hataları bastırmak.

### Agile/Scrum/Kanban metodolojileri

#### Agile (Çevik)
Agile, yazılım geliştirmede esnek, iteratif ve müşteri odaklı bir yaklaşımdır.
Büyük planlar yerine küçük parçalara bölerek sık geri bildirimle ilerlemeyi hedefler.

**Özellikleri:**
- Değişime hızlı uyum
- Kısa geliştirme döngüleri
- Sürekli geri bildirim

#### Scrum
Scrum, Agile'ın en yaygın kullanılan çerçevesidir (framework).
Geliştirme süreci Sprint adı verilen kısa zaman dilimlerine bölünür.

**Temel kavramlar:**
- Sprint
- Product Owner
- Scrum Master
- Daily Scrum

#### Kanban
Kanban, işlerin görsel bir pano üzerinden yönetildiği, akış odaklı bir yöntemdir. Sprint yoktur, işler sürekli ilerler.

**Özellikleri:**
- Kanban board (To Do – In Progress – Done)
- Aynı anda yapılan iş sayısını sınırlar (WIP)
- Sürekli teslimat

---

## 2. .NET Ekosistemi

### .NET nedir?

.NET, Microsoft tarafından geliştirilen, çok dilli, açık kaynak ve platform bağımsız bir yazılım geliştirme platformudur. Web, masaüstü, mobil, oyun ve bulut uygulamaları geliştirmek için kullanılır.

#### .NET'in tarihçesi
- 2000 → Microsoft tarafından duyuruldu.
- 2002 → İlk sürüm (.NET Framework 1.0) yayınlandı.
- 2016 → .NET Core çıktı. (açık kaynak ve cross-platform)
- 2020 → .NET Framework ve .NET Core birleşti → .NET 5
- Günümüzde → .NET 6 / 7 / 8 ile aktif olarak geliştirilmektedir.

#### .NET'in amacı
- Farklı platformlarda çalışan uygulamalar geliştirmek.
- Güvenli, performanslı ve ölçeklenebilir sistemler oluşturmak.
- Geliştirme sürecini kolaylaştırmak ve standartlaştırmak.

#### .NET neden kullanılır?
- C#, F#, VB.NET gibi dilleri destekler.
- Web (ASP.NET), masaüstü, mobil (MAUI) geliştirme imkânı sunar.
- Yüksek performans ve güvenlik sağlar.
- Geniş kütüphane ve topluluk desteği vardır.
- Azure ile güçlü entegrasyon sunar.

### .NET Framework, .NET Core ve .NET 7/8+ farkları

| Temel Alınan | .NET Core | .NET Framework | .NET 7 / 8+ |
|-------------|----------|----------------|-------------|
| Açık Kaynak | Açık kaynaklıdır. | Bazı bileşenleri açık kaynaklıdır. | Tamamen açık kaynaklıdır. |
| Çapraz Platform | Windows, Linux ve macOS destekler. | Sadece Windows üzerinde çalışır. | Windows, Linux, macOS (tam cross-platform). |
| Uygulama Modelleri | Web, API ve mikroservis odaklıdır. | Web ve masaüstü (WinForms, WPF). | Web, API, masaüstü, mobil (.NET MAUI), mikroservis. |
| Kurulum | Platformdan bağımsız paketlenir. | Windows için tek paket halinde kurulur. | Yan yana (side-by-side) kurulum destekler. |
| Mikroservis Desteği | Mikroservis mimarisini destekler. | Mikroservis mimarisi için uygun değildir. | Mikroservis ve cloud-native mimariler için idealdir. |
| REST API Desteği | REST API oluşturmayı destekler. | REST API desteği vardır. | REST API ve minimal API desteği vardır. |
| Performans | Yüksek performanslıdır. | Performansı .NET Core'a göre düşüktür. | Çok yüksek performans (en optimize sürüm). |
| Ölçeklenebilirlik | Yüksek ölçeklenebilirlik sunar. | Ölçeklenebilirlik sınırlıdır. | Cloud ve container ortamlarında yüksek ölçeklenebilirlik. |
| Uyumluluk | Windows, Linux, macOS | Sadece Windows | Windows, Linux, macOS |
| Android / Mobil | Xamarin üzerinden desteklenir. | Mobil geliştirme desteği yoktur. | .NET MAUI ile yerel mobil geliştirme desteklenir. |
| Paketleme | NuGet paketleri şeklinde sunulur. | Tüm kütüphaneler birlikte gelir. | NuGet + modüler yapı |
| Dağıtım Modeli | Yan yana sürüm çalıştırabilir. | Güncellemeler IIS ve sistem geneline etki eder. | Esnek, side-by-side ve container uyumlu. |
| WCF Hizmetleri | Tam WCF desteği yoktur. | WCF için güçlü destek sunar. | WCF önerilmez, modern alternatifler kullanılır. |
| CLI Araçları | Hafif ve güçlü CLI desteği vardır. | CLI desteği sınırlıdır. | Gelişmiş ve modern CLI araçları. |
| Güvenlik | Modern güvenlik yaklaşımları kullanır. | Kod Erişim Güvenliği (CAS) vardır. | Güncel ve gelişmiş güvenlik mekanizmaları. |
| Destek Durumu | Geliştirilmesi durduruldu. | Bakım modunda. | Aktif olarak geliştirilmektedir (LTS: .NET 8). |

### Platformlar arası çalışabilir mi?

Evet, .NET Core ve .NET 7/8+ platformlar arası çalışabilir (Windows, Linux, macOS), ancak .NET Framework sadece Windows üzerinde çalışır.

### dotnet --info çıktısı örneği ve yorumlama

```
.NET SDK:
 Version:           9.0.100-preview.7.24407.12
 Commit:            d672b8a045
 Workload version:  9.0.100-manifests.a8e34f65
 MSBuild version:   17.12.0-preview-24374-02+48e81c6f1
```

Version → Yüklü SDK sürümü. Burada 9.0.100-preview (önizleme sürüm).
MSBuild version → Projelerin derlenmesinde kullanılan araç.

```
Çalışma Zamanı Ortamı:
 OS Name:     Mac OS X
 OS Version:  26.2
 OS Platform: Darwin
 RID:         osx-arm64
 Base Path:   /usr/local/share/dotnet/sdk/9.0.100-preview.7.24407.12/
```

OS Name / OS Version / OS Platform → Çalıştığın işletim sistemi bilgisi.
RID → Runtime Identifier, burada osx-arm64, yani Apple M1/M2 işlemci tabanlı Mac.
Base Path → SDK'nın sistemde kurulu olduğu dizin.

```
.NET iş yükleri yüklendi:
Yeni bildirimler yüklenirken loose manifests kullanılacak şekilde yapılandırıldı.
Görüntülenecek yüklü iş yükü yok.

Host:
  Version:      9.0.0-preview.7.24405.7
  Architecture: arm64
  Commit:       static

.NET SDKs installed:
  6.0.404 [/usr/local/share/dotnet/sdk]
  6.0.405 [/usr/local/share/dotnet/sdk]
  7.0.101 [/usr/local/share/dotnet/sdk]
  7.0.102 [/usr/local/share/dotnet/sdk]
  8.0.201 [/usr/local/share/dotnet/sdk]
  9.0.100-preview.7.24407.12 [/usr/local/share/dotnet/sdk]
```

Sisteminde birden fazla SDK yüklü → farklı projeler farklı SDK'ları kullanabilir. Preview sürümü → en yeni özellikler deneme amaçlı.

```
.NET runtimes installed:
  Microsoft.AspNetCore.App 6.0.12 [/usr/local/share/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.AspNetCore.App 6.0.13 [/usr/local/share/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.AspNetCore.App 7.0.1 [/usr/local/share/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.AspNetCore.App 7.0.2 [/usr/local/share/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.AspNetCore.App 8.0.2 [/usr/local/share/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.AspNetCore.App 9.0.0-preview.7.24406.2 [/usr/local/share/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.NETCore.App 6.0.12 [/usr/local/share/dotnet/shared/Microsoft.NETCore.App]
  Microsoft.NETCore.App 6.0.13 [/usr/local/share/dotnet/shared/Microsoft.NETCore.App]
  Microsoft.NETCore.App 7.0.1 [/usr/local/share/dotnet/shared/Microsoft.NETCore.App]
  Microsoft.NETCore.App 7.0.2 [/usr/local/share/dotnet/shared/Microsoft.NETCore.App]
  Microsoft.NETCore.App 8.0.2 [/usr/local/share/dotnet/shared/Microsoft.NETCore.App]
  Microsoft.NETCore.App 9.0.0-preview.7.24405.7 [/usr/local/share/dotnet/shared/Microsoft.NETCore.App]
```

Runtimes, uygulamaların çalıştırıldığı ortamı gösterir. Birden fazla runtime yüklü → eski projeler eski sürümle çalışabilir.

```
Other architectures found:
  None

Environment variables:
  Not set

global.json file:
  Not found
```

Başka platform mimarisi yok. Ortam değişkenleri default.
global.json yok → SDK seçimi otomatik yapılır

### Senkron ve Asenkron Programlama

Senkron görevler sırayla gerçekleşir; bir sonraki göreve geçmeden önce ilk görevi bitirmeniz gerekir. Örneğin, bir alışveriş uygulamasını ele alalım. Kullanıcı siparişini görüntülediğinde, yazı tipi boyutu artmalıdır. Önce geçmişi yüklemeyi ve yazı tipi boyutunu güncellemeyi beklemek yerine, eşzamansız programlama her iki işlemin de aynı anda gerçekleşmesini sağlayabilir.

Öte yandan, asenkron görevleri herhangi bir sırayla veya hatta eş zamanlı olarak yürütebilirsiniz. Örneğin, kullanıcılar çevrimiçi alışveriş yaparken tüm ürünlerini tek tek değil, birlikte satın almak isterler. Kullanıcı sepete her ürün eklediğinde siparişi tamamlamak yerine, senkron programlama tüm ürünler için ödeme yönteminin ve gönderim adresinin aynı anda seçilmesini sağlar.

### async, await, Task, ConfigureAwait

**Async**: Bir metodun asenkron çalışacağını belirtir. Bu metod await ile çağrılabilir ve bir Task veya Task<T> döndürür.

```csharp
public async Task<int> GetirAsync() { ... }
```

**Await**: Asenkron bir işlemin bitmesini beklemek için kullanılır. Kodun geri kalanı bloklanmadan çalışmaya devam eder.

```csharp
public async Task<int> GetirAsync()
{
    await Task.Delay(1000);
    return 42;
}
```

**Task**: .NET'te asenkron işlemleri temsil eden bir nesnedir. void yerine Task veya Task<T> döndürür.

```csharp
public async Task<int> GetirAsync()
{
    await Task.Delay(1000);  
    return 42;               
}
```

**ConfigureAwait**: await'in hangi context'te devam edeceğini kontrol eder. Özellikle UI thread veya ASP.NET context'lerinde önemlidir.

```csharp
public async Task<int> GetirAsync()
{
    // false: UI context'e dönmeden arka planda devam et
    await Task.Delay(1000).ConfigureAwait(false);
    return 42;
}
```

### arrow function (=>) ifadesinin C#'taki yeri

| Kullanım Alanı | Örnek | Açıklama |
|-------------------------------|--------------------------------------|----------------------------------------|
| Lambda İfadeleri | `x => x * x` | Tek satırlık anonim fonksiyon |
| Expression-bodied Metot | `int Topla(int a, int b) => a + b;` | Tek satırda return |
| Expression-bodied Property | `public string AdSoyad => Ad + " " + Soyad;` | Tek satırlık property |
| LINQ / Koleksiyon İşlemleri | `sayilar.Select(x => x * x)` | Koleksiyon elemanları üzerinde kısa fonksiyon |

---

## 3. Backend Geliştirme Temelleri

### Backend nedir? Frontend ile farkları

Front-end, kullanıcının gördüğü ve etkileşimde bulunduğu her şeyden sorumluyken, Back-end bu işlemlerin arka planda nasıl gerçekleştiğiyle ilgilenir. Front-end geliştiricileri, görsel tasarım ve kullanıcı deneyimine odaklanırken, Back-end geliştiricileri veri işleme, güvenlik ve performans konularında çalışır.

### Web sunucusu nedir? API nedir? API türleri

**Web Sunucusu**: İnternet üzerinden gelen HTTP isteklerini alan ve yanıtlayan bilgisayar yazılımı veya sistem.

**API (Application Programming Interface)**: Uygulamalar veya sistemler arasında veri ve işlev paylaşımını sağlayan arayüz.

**API Türleri**: REST, SOAP, GraphQL, gRPC, WebSocket

### HTTP nedir? HTTP metodları

HTTP (Hypertext Transfer Protocol), istemci ve sunucu arasında veri transferini sağlayan protokoldür. Web tarayıcıları, mobil uygulamalar veya API istemcileri HTTP üzerinden veri alır ve gönderir.

| HTTP Metodu | Amaç / İşlevi | Örnek Kullanım | Açıklama |
|-------------|---------------|----------------|----------|
| GET | Veri okuma | `/products` → Tüm ürünleri getir<br>`/products/123` → ID 123 olan ürünü getir | Request body içermez. Sadece veri almak için kullanılır. |
| POST | Yeni veri oluşturma | `/products` → Yeni ürün ekler<br>Request body: `{ "name":"Sneakers", "color":"blue", "price":59.95, "currency":"USD" }` | Request body ile gönderilen veriler sunucuda yeni kaynak oluşturur. |
| PUT | Mevcut veriyi güncelleme / değiştirme | `/products/123` → ID 123 olan ürünü tamamen değiştirir | Body'de belirtilmeyen alanlar silinir; yeni alanlar eklenir. |
| DELETE | Veri silme | `/products/123` → ID 123 olan ürünü siler | Yetkilendirme gerekebilir; kaynak kalıcı olarak silinir. |

### RESTful servislerin çalışma mantığı

REST (Representational State Transfer), web servislerini kaynak (resource) temelli ve stateless (durumsuz) olarak tasarlamak için kullanılan bir mimari tarzıdır.
RESTful servisler, HTTP protokolünü kullanarak istemci ve sunucu arasında veri paylaşır.

### JSON veri formatı ve kullanım amacı

JSON (JavaScript Object Notation), veri depolamak ve taşımak için kullanılan basit, bağımsız bir veri değişim formatıdır. Amacı, sistemler arasında veri alışverişini standart ve anlaşılır bir biçimde sağlamak.

**Örnek Kullanım**: Web uygulaması ile sunucu arasında veri iletmek, API yanıtlarını göndermek veya almak, konfigürasyon dosyalarını saklamak

```json
{
  "isim": "Ahmet",
  "yas": 25,
  "sehir": "İstanbul"
}
```

### SOAP ve GraphQL nedir, REST'ten farkları

GraphQL API'ler için tek uç nokta üzerinden çalışan bir sorgu dilidir. İstemciler sadece ihtiyaç duydukları veriyi talep edebilir, iç içe sorguları destekler ve şema üzerinden veri tiplerini tanımlar. Gerçek zamanlı güncellemeler ve abonelikler ile modern web ve mobil uygulamalar için esneklik sağlar.

SOAP ise XML tabanlı bir mesajlaşma protokolüdür ve yüksek güvenlik, güvenilirlik ile kurumsal uyumluluk gerektiren sistemlerde kullanılır. WSDL ile resmi istemci-sunucu sözleşmesi tanımlar, durumlu veya durumsuz işlemleri destekler ve WS-Security ile mesaj düzeyinde şifreleme, imza ve kimlik doğrulamayı sağlar.

#### REST API, GraphQL ve SOAP Karşılaştırma Tablosu

| Bakış Açısı | REST API | GraphQL | SOAP |
|-------------|----------|---------|------|
| Veri Formatı | JSON / XML | Şemaya göre yapılandırılmış JSON | XML |
| Uç Nokta | Birden fazla kaynağa özgü URL | Tek uç nokta (`/graphql`) | WSDL aracılığıyla tanımlanan tek uç nokta |
| Sorgu Esnekliği | Sabit yapı, fazla/eksik veri çekme olasılığı | Esnek, istemci ihtiyaç duyduğu verileri belirler | Katı, sıkı istek/yanıt yapısı |
| Gerçek Zamanlı Destek | Sınırlı (sorgulama / web kancaları) | Abonelikler desteklenir | Dahili olarak mevcut değil, ek protokollerle mümkün |
| İşlem Desteği | Doğal olarak değil (Sagas gibi geçici çözümler) | Kalıtsal değil; özel mantık gerekebilir | Dahili (WS-AtomicTransaction) |
| Güvenlik | Taşıma seviyesi (HTTPS, OAuth, JWT) | REST ile aynı; dikkatli yetkilendirme gerekir | Güçlü—WS-Security (şifreleme, imzalar) |
| Şema ve Sözleşme | Genellikle gevşek; OpenAPI ile belgelenir | Güçlü tipli şema, iç gözlem | WSDL sözleşmesi ile katı |
| Performans | Basit durumlar için verimli | Verimli; sorgulama maliyeti kontrol edilmeli | Ayrıntılı XML nedeniyle ağır |
| Araçlar ve Topluluk | Kapsamlı (Postman, Insomnia) | Hızla büyüyor (Apollo, GraphiQL) | İstikrarlı; kurumsal odaklı (SoapUI) |
| Sürümleme | URL veya başlıklarla sürümlendirme | Sürüm yok; eski özellikler kullanımdan kaldırılır | WSDL'ye bağlı; evrimleşmesi zor |
| Popüler Kullanım Alanları | Web/mobil uygulamalar, halka açık API'ler, mikro hizmetler | Kontrol panelleri, mobil arayüzler, gerçek zamanlı sistemler | Bankacılık, telekomünikasyon, sağlık hizmetleri entegrasyonları |
| Örnek Kullanım | Twitter, GitHub, Stripe | Facebook, Shopify, GitHub (v4) | PayPal, eski kurumsal sistemler |

---

## 4. ASP.NET

### ASP.NET ve ASP.NET Core nedir?

ASP.NET, Microsoft tarafından geliştirilmiş, Windows tabanlı web uygulamaları ve API'ler oluşturmak için kullanılan web framework'üdür.

ASP.NET Core, ASP.NET'in yeni, açık kaynak ve platformlar arası sürümüdür. Windows, Linux ve macOS üzerinde çalışabilir. Daha hızlı ve hafif bir yapıya sahiptir.

#### ASP.NET vs ASP.NET Core Karşılaştırması

| Özellik | ASP.NET | ASP.NET Core |
|---------|---------|--------------|
| Tanım | Microsoft'un Windows tabanlı web uygulamaları ve API framework'ü | ASP.NET'in tamamen yeniden tasarlanmış, açık kaynak ve platformlar arası sürümü |
| Uyumluluk | Windows sunucularında en iyi performans, Linux üzerinde sınırlı destek | Windows, Linux ve macOS sunucularında çalışabilir |
| Performans | Orta, ağır ve monolitik yapı | Modüler, hafif ve yüksek performans; platformlar arası ölçeklenebilir |
| Ekosistem | 20 yıllık geçmiş, geniş kütüphane ve topluluk desteği | Hızla büyüyen ekosistem, modern web geliştirme için optimize edilmiş |
| Destek | Yakında yalnızca güvenlik güncellemeleri alacak; yeni özellik geliştirilmez | Microsoft tarafından aktif olarak geliştirilmekte ve desteklenmekte |
| Modern Web Desteği | Eski tip web uygulamaları, Web Forms, MVC | Web API, Razor Pages, Blazor, mikroservisler için ideal |
| Hosting | IIS üzerinde | IIS, Nginx, Apache veya kendi sunucusu üzerinde |
| Mikroservis Desteği | Sınırlı | Tam destek |

### MVC nedir, ne için kullanılır?

MVC (Model-View-Controller), yazılım geliştirmede kullanılan bir tasarım desenidir. Uygulamanın üç ana bileşene ayrılmasını sağlar:

- **Model**: Veri ve iş mantığı (veritabanı, hesaplamalar)
- **View**: Kullanıcıya gösterilen arayüz (UI)
- **Controller**: Model ve View arasındaki iletişimi yöneten katman

MVC, özellikle web uygulamaları ve API'lerde sık kullanılır ve temiz, test edilebilir kod yapısı sağlar.

### Middleware nedir, nasıl çalışır?

Middleware, farklı uygulamalar, sistemler ve veritabanları arasında etkileşim ve veri akışı sağlayan bulut hizmetleridir. Uygulamalar ve veriler arasında bir köprü görevi görür. Bu sayede sistemler arasında kesintisiz iletişim sağlar ve veri akışını düzenler.

**Middleware Nasıl Çalışır?**

HTTP isteği uygulamaya geldiğinde, middleware zinciri üzerinden geçer.
Her middleware, isteği işleyebilir, değiştirebilir veya zincirdeki sonraki middleware'e iletebilir. Yanıt gönderilirken, middleware zincirinden ters yönde geçer ve yanıt üzerinde işlem yapabilir.

### Startup.cs ya da Program.cs içindeki middleware sıralaması

ASP.NET Core, HTTP isteklerini middleware zinciri (pipeline) üzerinden işler.
Middleware'ler sırayla çalışır: sıra değişirse uygulama davranışı değişebilir.
Yanlış sırada eklenen middleware, isteklerin düzgün işlenmemesine, yetkilendirme hatalarına veya hata yakalama sorunlarına yol açabilir.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Hata yakalama middleware (en üstte olmalı)
app.UseExceptionHandler("/Home/Error");

// Statik dosya middleware
app.UseStaticFiles();

// Routing middleware
app.UseRouting();

// Yetkilendirme ve kimlik doğrulama
app.UseAuthentication();
app.UseAuthorization();

// Endpoint tanımları (en sonda)
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
});

app.Run();
```

### Dependency Injection (DI) nedir, neden önemlidir?

Dependency Injection, bir sınıfın ihtiyaç duyduğu bağımlılıkları (diğer sınıflar, servisler veya nesneler) kendisi oluşturmak yerine dışarıdan alması prensibidir.
Gevşek bağlı, test edilebilir ve esnek uygulamalar geliştirmek için ASP.NET Core'da standart olarak kullanılır.

**Örnek:**

```csharp
// 1. Servis Tanımı
public interface IMessageService
{
    string GetMessage();
}

// Servisin implementasyonu
public class EmailService : IMessageService
{
    public string GetMessage()
    {
        return "Email gönderildi!";
    }
}

// 2. DI Konfigürasyonu (Program.cs veya Startup.cs)
var builder = WebApplication.CreateBuilder(args);

// IMessageService bağımlılığı EmailService ile eşleniyor
builder.Services.AddScoped<IMessageService, EmailService>();

var app = builder.Build();

// 3. Controller'da Kullanımı
public class HomeController : Controller
{
    private readonly IMessageService _messageService;

    // DI sayesinde EmailService otomatik olarak inject edilir
    public HomeController(IMessageService messageService)
    {
        _messageService = messageService;
    }

    public IActionResult Index()
    {
        var msg = _messageService.GetMessage();
        return Content(msg); // Çıktı: "Email gönderildi!"
    }
}
```

### Katmanlı Mimari (Layered Architecture)

#### 1. Presentation Layer
- Kullanıcı arayüzünü ve istemci isteklerini yönetir
- Controller, View, API endpoint'leri burada yer alır
- İş mantığı içermez, yalnızca Business Layer ile iletişim kurar

#### 2. Business Layer
- Uygulamanın iş kuralları ve iş mantığını içerir
- Verilerin nasıl işleneceğine karar verir
- Presentation ile Data Access arasında köprü görevi görür

#### 3. Data Access Layer
- Veritabanı işlemlerini yönetir (CRUD)
- ORM (Entity Framework Core vb.) burada kullanılır
- Verinin nasıl saklandığıyla ilgilenir

#### Service & Repository Pattern

**Service Pattern**
- İş mantığını controller'dan ayırmak için kullanılır
- Controller → Service → Repository akışı sağlar
- Kod tekrarını azaltır, test edilebilirliği artırır

**Repository Pattern**
- Veritabanı erişimini soyutlamak için kullanılır
- Business katmanının veritabanı detaylarını bilmesini engeller
- ORM veya veritabanı değişikliklerini kolaylaştırır

### Katmanlar arası ilişki

Bağımlılıklar içten dışa doğrudur; Domain katmanı bağımsızdır.
Application, Domain ile etkileşir.
Infrastructure, Domain ve Application katmanları için dış sistemleri sağlar.
API/Presentation, Application ve Domain üzerinden veri alır ve kullanıcıya sunar.

| Özellik | Katmanlı Mimari | Clean Architecture |
|---------|----------------|------------------|
| Bağımlılık Yönü | Üstten alta (Presentation → Data Access) | İçten dışa (Domain bağımsız) |
| Test Edilebilirlik | Orta | Yüksek |
| Esneklik | Orta | Yüksek |
| Kullanım | Küçük/orta web uygulamaları | Büyük, karmaşık ve uzun ömürlü uygulamalar |

### Clean Architecture

#### 1. Domain Layer
- Uygulamanın çekirdeğidir
- Temel iş kuralları ve domain modelleri bulunur
- Hiçbir dış katmana bağımlı değildir

#### 2. Application Layer
- Use-case'ler, servisler ve iş akışları burada yer alır
- Domain katmanını kullanır
- Altyapı detaylarını bilmez

#### 3. Infrastructure Layer
- Veritabanı, dosya sistemi, dış servisler, e-posta, cache vb.
- Application ve Domain için gerekli teknik detayları sağlar
- Repository ve dış servis implementasyonları burada bulunur

#### 4. API / Presentation Layer
- Kullanıcı veya istemcilerle iletişim kurar
- Controller, API endpoint'leri, UI bileşenleri
- Application katmanını çağırır

#### Bağımlılıkların Dışa Akması (Dependency Rule)

- Bağımlılıklar her zaman içe doğru olmalıdır
- İç katmanlar dış katmanları bilmez
- Dış katmanlar iç katmanlara bağımlıdır

---

## 5. Veritabanı ve ORM

### SQL nedir?

SQL, yapılandırılmış veritabanlarıyla iletişim kurmak için kullanılan standart bir sorgulama dilidir. Veritabanlarından veri ekleme, okuma, güncelleme ve silme (CRUD) işlemlerini yapar.

### İlişkisel ve ilişkisel olmayan veritabanları arasındaki farklar

| Özellik | İlişkisel Veritabanı (RDBMS) | İlişkisel Olmayan Veritabanı (NoSQL) |
|---------|------------------------------|-------------------------------------|
| Veri Yapısı | Tablo, satır ve sütun | JSON, key-value, document, graph vb. |
| Şema | Sabit, önceden tanımlanmış | Esnek, dinamik şema |
| Örnek Sistemler | MySQL, PostgreSQL, SQL Server, Oracle | MongoDB, Cassandra, Redis, CouchDB |
| Sorgulama Dili | SQL | API veya sorgu dili (MongoDB Query, CQL) |
| İlişkiler | Tablo ilişkileri ile (JOIN) | Genellikle ilişki yok veya sınırlı |
| Ölçeklenebilirlik | Dikey (daha güçlü sunucu) | Yatay (sunucu ekleyerek) |
| Performans | Karmaşık ilişkilerde güçlü | Büyük veri ve yüksek trafikte optimize |
| Uygulama Alanı | Bankacılık, muhasebe, kurumsal uygulamalar | Sosyal medya, gerçek zamanlı veri, IoT, büyük veri |
| Veri Tutarlılığı | ACID (Atomicity, Consistency, Isolation, Durability) | BASE (Basically Available, Soft state, Eventually consistent) |

### ORM nedir? Entity Framework Core nedir?

Object Relational Mapping (Nesne İlişkisel Eşleme) olarak açılımını yapabileceğimiz OOP tabanlı programlama dillerinde veritabanı tabloları ile bu tabloları temsil eden nesnelerin eşlenmesini ve bu nesneler üzerinden veritabanı işlemlerini gerçekleştirirken ihtiyaç duyulan sorguların otomatik generate edilmesini sağlayan bir yaklaşımdır.

Entity Framework, Microsoft'un .NET Core ve .NET 5/6+ için geliştirdiği açık kaynak ORM'idir.
Hem RDBMS ile çalışır (SQL Server, PostgreSQL, MySQL) hem de LINQ ile veri sorgulama sağlar.

### DbContext nedir, nasıl kullanılır?

DbContext, Entity Framework (EF / EF Core) ile veritabanı işlemlerini yöneten ana sınıftır. Amacı, veritabanı bağlantısı kurmak, tabloları (DbSet'leri) yönetmek ve CRUD işlemlerini nesne odaklı şekilde gerçekleştirmek.

### LINQ nedir? En çok kullanılan LINQ ifadeleri

LINQ (Language Integrated Query), Microsoft tarafından geliştirilen bir programlama tekniğidir. LINQ, .NET programlama dillerinde (C#, Visual Basic, vb.) veri koleksiyonları üzerinde sorgulama ve manipülasyon yapmayı kolaylaştıran bir yol sunar. En sık kullanılanlar: Where, Select, FirstOrDefault, OrderBy, GroupBy, Any, All, Count, Sum, Average.

| LINQ Örneği | Açıklama | SQL Karşılığı |
|-------------|----------|---------------|
| `var adults = people.Where(p => p.Age >= 18);` | Yaşı 18 ve üzeri kişileri filtreler | `SELECT * FROM People WHERE Age >= 18;` |
| `var names = people.Select(p => p.Name);` | Sadece isimleri seçer | `SELECT Name FROM People;` |
| `var sorted = people.OrderBy(p => p.Age);` | Yaşa göre artan sıralama | `SELECT * FROM People ORDER BY Age ASC;` |
| `var totalSalary = employees.Sum(e => e.Salary);` | Maaş toplamını hesaplar | `SELECT SUM(Salary) FROM Employees;` |

### Code-First ve Database-First yaklaşımı nedir?

Code-First, önce C# sınıfları (model) yazılır, EF Core bu sınıflardan veritabanını oluşturur.
Database-First, önce veritabanı tasarlanır, EF Core modelleri veritabanından oluşturulur.

| Özellik | Code-First | Database-First |
|---------|------------|----------------|
| Tanım | Önce C# sınıfları yazılır, veritabanı bu sınıflardan oluşturulur | Önce veritabanı tasarlanır, modeller veritabanından üretilir |
| Geliştirme Akışı | Kod → Veritabanı | Veritabanı → Kod |
| Avantaj | Hızlı prototipleme<br>Kod odaklı geliştirme<br>Migration ile şema güncellemeleri kolay | Mevcut veritabanı ile uyumlu<br>Kurumsal projeler için ideal |
| Dezavantaj | Mevcut büyük veritabanları için uygun değil | Koddan bağımsız, şema değişiklikleri zor |
| Kullanım Alanı | Yeni projeler, hızlı geliştirme | Mevcut veritabanı, büyük kurumsal uygulamalar |

### Temel SQL sorguları

**SELECT**: Veritabanındaki verileri sorgulamak için kullanılır. Bu komut veritabanındaki belirli bir tablodan belirli bir sütunu seçmek için kullanılır.

```sql
SELECT description,name,price,quantity FROM products;
```

**INSERT INTO**: Veritabanındaki tablolarımıza yeni veriler eklemek için kullanılır.

```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

**UPDATE**: Veritabanındaki bir kaydı güncellemek için kullanılır.

```sql
UPDATE products SET price = 38000 WHERE id = 11;
```

**DELETE**: Veritabanındaki verileri silmemizi sağlar.

```sql
DELETE FROM products WHERE id = 11;
```

---

## 6. Güvenlik ve Performans

### Authentication vs Authorization nedir?

**Authentication (Kimlik Doğrulama)**, bir kullanıcının gerçekten iddia ettiği kişi olup olmadığını kontrol etme sürecidir. Bu aşamada sistem, kullanıcının kim olduğunu doğrular. Genellikle kullanıcı adı–şifre, JWT token, OAuth veya tek oturum açma (SSO) gibi yöntemler kullanılır. Authentication, uygulamaya giriş (login) sırasında gerçekleşir.

**Authorization (Yetkilendirme)** ise kimliği doğrulanmış bir kullanıcının hangi işlemleri yapabileceğini belirleme sürecidir. Kullanıcının belirli bir kaynağa erişip erişemeyeceği, rolü veya sahip olduğu izinlere göre kontrol edilir. Örneğin bir kullanıcının yalnızca veri görüntüleyebilmesi, ancak silme işlemini yapamaması authorization ile belirlenir.

### JWT (JSON Web Token) nedir, nasıl çalışır?

JWT (JSON Web Token), istemci ile sunucu arasında kimlik doğrulama ve yetkilendirme bilgilerini güvenli şekilde taşımak için kullanılan, kompakt ve durumsuz (stateless) bir token formatıdır. Özellikle RESTful API'lerde yaygın olarak kullanılır.

JWT'nin çalışma sürecinde kullanıcı sisteme giriş yapar, sunucu bilgileri doğrular ve bir JWT üretir. Bu token istemciye gönderilir ve istemci sonraki her istekte token'ı sunucuya iletir. Sunucu token'ı doğrular; geçerliyse isteğe izin verir, geçersizse erişimi reddeder. Böylece sunucu tarafında oturum bilgisi tutulmasına gerek kalmaz.

JWT üç temel bileşenden oluşur. **Header**, token'ın türünü (JWT) ve kullanılan imzalama algoritmasını belirtir. **Payload**, kullanıcıya veya oturuma ait bilgileri (claim'leri) içerir. **Signature** ise token'ın değiştirilmediğini ve güvenilir bir kaynaktan geldiğini doğrulamak için kullanılır.

### OAuth, OAuth2.0, OpenIddict, OpenID nedir?

**OAuth**: Yetkilendirme (authorization) için ortaya çıkan genel bir kavramdır. Bir uygulamanın, kullanıcı şifresini almadan başka bir servis adına işlem yapabilmesini amaçlar.

**OAuth 2.0**: OAuth'un güncel ve yaygın kullanılan versiyonudur. "Giriş yapmak" için değil, erişim izni vermek içindir. Token (access token) mantığıyla çalışır.

**OpenID**: Eski bir kimlik doğrulama (authentication) standardıdır. "Kullanıcı kim?" sorusuna cevap verir. Günümüzde büyük ölçüde terk edilmiştir.

**OpenID Connect (OIDC)**: OAuth 2.0 üzerine kurulmuş modern bir kimlik doğrulama katmanıdır. OAuth'a "login olma" yeteneğini ekler. ID Token (JWT) kullanır.

**OpenIddict**: OAuth 2.0 ve OpenID Connect'i uygulamak için kullanılan bir .NET kütüphanesidir. Yani bir standart değil, bu standartları hayata geçirmenizi sağlar.

### Performans artımı için ne yapılabilir?

Gereksiz takip ve bellek kullanımını azaltmak, veriyi asenkron ve parça parça işlemek, sık kullanılan verileri cache'lemek ve darboğazları ölçerek optimize etmek performansı artırır.

**AsNoTracking**: Entity Framework Core'da sadece veri okumak için yapılan sorgularda kullanılır. Change tracking kapandığı için bellek kullanımı azalır ve sorgular daha hızlı çalışır.

**IAsyncEnumerable**: Verileri tek seferde belleğe almak yerine parça parça (streaming) okumayı sağlar. Büyük veri setlerinde bellek tüketimini düşürür ve daha hızlı ilk sonuç alınmasına yardımcı olur.

**Caching**: Sık kullanılan ve nadiren değişen verilerin tekrar tekrar veritabanından çekilmesini önler. Bellek içi cache (MemoryCache) veya dağıtık cache kullanılarak yanıt süreleri ciddi şekilde düşürülür.

**Profiling**: Uygulamanın hangi noktalarının yavaş çalıştığını tespit etmek için kullanılır. Gereksiz sorgular, yavaş metodlar ve darboğazlar tespit edilerek hedefli optimizasyon yapılmasını sağlar.

**Redis**: Dağıtık ve çok hızlı bir in-memory cache sistemidir. Özellikle birden fazla sunucu kullanılan sistemlerde cache'in ortak kullanılmasını sağlar ve veritabanı yükünü büyük ölçüde azaltır.

### OWASP 10 listesi

**Broken Access Control**: Kullanıcıların yetkileri doğru sınırlandırılmazsa, başkalarına ait verilere veya işlemlere erişebilirler.

**Cryptographic Failures**: Hassas verilerin şifrelenmemesi veya zayıf kriptografi kullanılması veri sızıntılarına yol açar.

**Injection**: Kötü niyetli girdiler (SQL, NoSQL, OS komutları) uygulama tarafından çalıştırılabilir.

**Insecure Design**: Güvenlik, uygulama tasarım aşamasında yeterince düşünülmemiştir.

**Security Misconfiguration**: Yanlış yapılandırmalar (açık debug modları, varsayılan ayarlar) sistemi savunmasız bırakır.

**Vulnerable and Outdated Components**: Güncel olmayan veya güvenlik açığı olan kütüphaneler kullanılır.

**Identification and Authentication Failures**: Kimlik doğrulama ve oturum yönetimi hataları hesap ele geçirmeye neden olur.

**Software and Data Integrity Failures**: Güvenilmeyen kaynaklardan gelen kod veya veriler doğrulanmadan kullanılır.

**Security Logging and Monitoring Failures**: Güvenlik olayları yeterince loglanmaz ve izlenmez, saldırılar fark edilmez.

**Server-Side Request Forgery (SSRF)**: Sunucu, saldırganın yönlendirdiği isteği iç ağ veya hassas servislere gönderir.

### Web uygulamalarında en yaygın güvenlik açıkları

**SQL Injection**: Kullanıcı girdileriyle veritabanı sorguları manipüle edilir.

**XSS (Cross-Site Scripting)**: Tarayıcıda zararlı JavaScript kodu çalıştırılır.

**CSRF (Cross-Site Request Forgery)**: Kullanıcı haberi olmadan yetkili işlemler yaptırılır.

**Broken Authentication**: Hatalı kimlik doğrulama nedeniyle hesaplar ele geçirilir.

**Broken Access Control**: Yetkisiz kullanıcılar kısıtlı kaynaklara erişir.

### ASP.NET Core ile alınabilecek önlemler

**Model Validation**: [Required], [StringLength] gibi attribute'larla hatalı/veri girişleri engellenir.

**Input Sanitization**: Zararlı karakterler temizlenerek XSS ve Injection riskleri azaltılır.

**Authentication & Authorization**: JWT, Identity ve [Authorize] ile erişim kontrolü sağlanır.

**Anti-Forgery (CSRF)**: ValidateAntiForgeryToken ile sahte istekler engellenir.

**HTTPS Zorunluluğu**: UseHttpsRedirection ile veriler şifreli iletilir.

---

## 7. Logging ve Hata Yönetimi

### Neden loglama yapılır? Log seviyesi nedir?

Loglama; uygulamadaki hataları tespit etmek, sistemi izlemek, performans sorunlarını analiz etmek ve güvenlik olaylarını takip edebilmek için yapılır. Üretim ortamında sorunların nedenini anlamanın en temel yoludur.

Log seviyesi, kaydedilecek logların önem derecesini belirtir. En yaygın seviyeler:

- **Trace**: En detaylı, geliştirme amaçlı
- **Debug**: Hata ayıklama için
- **Information**: Normal uygulama akışı
- **Warning**: Olası sorunlar
- **Error**: Çalışma zamanı hataları
- **Critical**: Sistemi durdurabilecek hatalar

### ASP.NET Core'da logging altyapısı

ASP.NET Core'da logging altyapısı, uygulama içindeki olayları ve hataları merkezi, esnek ve genişletilebilir bir yapı ile kaydetmek için kullanılır. Framework, yerleşik (built-in) bir logging sistemiyle gelir ve farklı log sağlayıcılarını (provider) destekler. Logging altyapısının temeli ILogger, ILogger<T> ve ILoggerFactory arayüzlerine dayanır. Genellikle ILogger<T> Dependency Injection (DI) ile sınıflara enjekte edilerek kullanılır.

### Global exception handling nasıl yapılır?

Global exception handling, uygulama genelinde oluşan hataları tek bir noktadan yakalayıp yönetmeyi sağlar. ASP.NET Core'da genellikle middleware kullanılarak yapılır. Program.cs içinde eklenen UseExceptionHandler veya özel bir Exception Handling Middleware, fırlatılan hataları yakalar, loglar ve kullanıcıya kontrollü bir hata cevabı (örneğin 500) döner. Geliştirme ortamında ise UseDeveloperExceptionPage detaylı hata ekranı gösterir. Bu yaklaşım kod tekrarını azaltır, güvenliği artırır ve hataların merkezi olarak yönetilmesini sağlar.

### UseExceptionHandler ve ILogger nasıl kullanılır?

**UseExceptionHandler**: Uygulama genelindeki yakalanmayan hataları tek noktadan yakalar

**ILogger**: Hatayı loglar (console, dosya, vs.)

```csharp
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var exception = context.Features.Get<IExceptionHandlerFeature>()?.Error;
        var logger = context.RequestServices.GetRequiredService<ILogger<Program>>();

        logger.LogError(exception, "Global bir hata oluştu");

        context.Response.StatusCode = 500;
        await context.Response.WriteAsync("Sunucu hatası oluştu.");
    });
});
```

---

## 8. Yazılım Geliştirme Prensipleri

### SOLID prensipleri

SOLID yazılım prensipleri, esnek, sürdürülebilir ve anlaşılır yazılımlar geliştirmeyi amaçlayan temel tasarım prensipleridir. Kod tekrarını azaltır, değişikliklere uyumu kolaylaştırır ve bakım maliyetini düşürerek "iyi kod" yazılmasını sağlar.

**S – Single Responsibility Principle**: Bir sınıfın yalnızca tek bir sorumluluğu olmalı ve sadece bu sorumluluk değiştiğinde güncellenmelidir.

**O – Open/Closed Principle**: Sınıflar değişikliğe kapalı, geliştirmeye açık olmalıdır; mevcut kod değiştirilmeden yeni özellikler eklenebilmelidir.

**L – Liskov Substitution Principle**: Alt sınıflar, üst sınıfların yerine kodda değişiklik gerektirmeden kullanılabilmelidir.

**I – Interface Segregation Principle**: Tek ve büyük arayüzler yerine, amaca özel küçük arayüzler tercih edilmelidir.

**D – Dependency Inversion Principle**: Üst seviye sınıflar, alt seviye sınıflara değil soyutlamalara bağımlı olmalıdır.

### Design Patterns: Singleton, Repository, Factory

Design Patterns (Tasarım Kalıpları) yazılımda sık karşılaşılan problemlere tekrar kullanılabilir çözümler sunar.

**Singleton**: Bir sınıftan uygulama boyunca yalnızca tek bir nesne oluşturulmasını sağlar ve bu nesneye her yerden erişilmesine imkân tanır.

**Repository**: Veri erişim işlemlerini uygulamanın iş mantığından ayırarak, daha temiz, test edilebilir ve sürdürülebilir bir yapı sunar.

**Factory**: Nesne oluşturma sürecini merkezileştirir ve hangi nesnenin üretileceğini istemciden gizleyerek esnek ve bağımlılığı düşük bir yapı sağlar.

### Clean Code nedir, neden önemlidir?

Clean Code, okunması kolay, anlaşılır, bakımı ve geliştirilmesi rahat olan koddur. İyi isimlendirme, küçük ve tek sorumluluklu metotlar ile tekrar etmeyen sade bir yapı hedeflenir. Örneğin x yerine totalPrice gibi ne yaptığını açıkça ifade eden değişken isimleri kullanmak, kodun daha anlaşılır olmasını sağlar. Clean Code, hata oranını azaltır, ekip içi anlaşılabilirliği artırır ve yazılımın uzun vadede sürdürülebilir olmasına katkı sağlar.

### Yazılım Mimari Desenleri

Yazılım Mimari Desenleri, uygulamanın yapısını düzenlemek, bağımlılıkları yönetmek ve ölçeklenebilirliği artırmak için kullanılan yaklaşımlardır:

**Layered Architecture**: Uygulama; Presentation, Business ve Data Access gibi katmanlara ayrılır. Her katman yalnızca bir alt katmanla iletişim kurar, bu da düzenli ve bakımı kolay bir yapı sağlar.

**Clean Architecture**: İş kuralları merkezdedir. Domain ve Application katmanları dış bağımlılıklardan izole edilir, böylece test edilebilirlik ve sürdürülebilirlik artar.

**Microservices**: Uygulama, bağımsız çalışabilen küçük servislerden oluşur. Her servis kendi verisini yönetir ve ölçeklenebilirlik sağlar.

**Event-Driven Architecture**: Bileşenler olaylar (event) üzerinden haberleşir. Gevşek bağlı yapı sunar ve asenkron sistemler için uygundur.

**Hexagonal Architecture (Ports & Adapters)**: İş mantığı dış dünyadan soyutlanır. Veritabanı, UI ve dış servisler adapter'lar aracılığıyla bağlanır, böylece bağımlılıklar azaltılır.

| Mimari | Yapı | Avantaj | Dezavantaj | Ne Zaman Tercih Edilir |
|------|------|--------|-----------|----------------------|
| Layered Architecture | Katmanlı | Basit ve anlaşılır | Katmanlar arası bağımlılık | Küçük–orta ölçekli CRUD uygulamalar |
| Clean Architecture | Domain merkezli | Test edilebilir, sürdürülebilir | Öğrenme maliyeti | Uzun vadeli ve büyüyen projeler |
| Microservices | Servis tabanlı | Yüksek ölçeklenebilirlik | Dağıtık sistem karmaşıklığı | Büyük ve yüksek trafikli sistemler |
| Event-Driven | Olay tabanlı | Gevşek bağlı yapı | Debug zor | Gerçek zamanlı ve asenkron sistemler |
| Hexagonal Architecture | Ports & Adapters | Düşük bağımlılık | Tasarım karmaşık | Domain odaklı, esnek sistemler |

### Hangi senaryoda hangi mimari tercih edilir?

Küçük veya orta ölçekli, basit CRUD uygulamaları için **Layered Architecture** yeterlidir.

Uzun vadeli, sık değişen iş kuralları olan projelerde **Clean Architecture** tercih edilir.

Yüksek trafik, bağımsız geliştirme ve ölçeklenebilirlik gerektiren sistemlerde **Microservices** uygundur.

Asenkron iletişim, bildirim, loglama veya gerçek zamanlı veri akışı gereken durumlarda **Event-Driven Architecture** kullanılır.

İş mantığının veritabanı, UI veya dış servislerden bağımsız olması isteniyorsa **Hexagonal Architecture** tercih edilir.
