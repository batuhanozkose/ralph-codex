# Ralph - Yeni Özellik Önerileri

Bu doküman, Ralph projesinin analizi sonucunda önerilen yeni özellikleri içermektedir.

---

## 🎯 Öncelikli Özellikler (Yüksek Etki)

### 1. **Dashboard ve İstatistikler**
Mevcut durumu görselleştiren bir web dashboard'u.

**Özellikler:**
- Toplam story sayısı, tamamlanan, bekleyen
- Iterasyon başına tamamlama oranı
- Süre istatistikleri (story başına ortalama süre)
- Son 10 iterasyonun özeti
- Gerçek zamanlı durum göstergesi

**Teknik Uygulama:**
- Flowchart klasöründeki React uygulamasına yeni bir sayfa eklenebilir
- `prd.json` ve `progress.txt` dosyalarını parse eden bir API endpoint'i
- Chart.js veya Recharts ile grafikler

**Dosya Değişiklikleri:**
```
flowchart/src/
├── pages/
│   ├── Dashboard.tsx (yeni)
│   └── Flowchart.tsx (mevcut App.tsx'den ayrılacak)
├── components/
│   ├── StatsCard.tsx
│   ├── ProgressChart.tsx
│   └── StoryList.tsx
└── utils/
    └── parseProgress.ts
```

---

### 2. **Slack/Discord Bildirimleri**
Ralph iterasyonları hakkında bildirim gönderme.

**Özellikler:**
- Story tamamlandığında bildirim
- Hata oluştuğunda alert
- Tüm PRD tamamlandığında başarı mesajı
- Max iteration'a ulaşıldığında uyarı

**Teknik Uygulama:**
```bash
# ralph.sh'e eklenecek
RALPH_SLACK_WEBHOOK="https://hooks.slack.com/..."
RALPH_DISCORD_WEBHOOK="https://discord.com/api/webhooks/..."

notify() {
  local message="$1"
  if [ -n "$RALPH_SLACK_WEBHOOK" ]; then
    curl -X POST -H 'Content-type: application/json' \
      --data "{\"text\":\"$message\"}" "$RALPH_SLACK_WEBHOOK"
  fi
}
```

---

### 3. **Retry Mekanizması ve Akıllı Hata Yönetimi**
Başarısız olan story'leri yeniden deneme.

**Özellikler:**
- Bir story 3 kez başarısız olursa otomatik skip
- Hata nedenlerini `progress.txt`'e kaydet
- `prd.json`'a `failCount` ve `lastError` alanları ekle
- Skip edilen story'leri raporla

**prd.json Güncellemesi:**
```json
{
  "id": "US-001",
  "title": "...",
  "passes": false,
  "failCount": 2,
  "lastError": "TypeScript compilation failed",
  "skipped": false
}
```

---

### 4. **Paralel Story Execution**
Bağımsız story'leri paralel çalıştırma.

**Özellikler:**
- `prd.json`'da `dependsOn` alanı
- Bağımsız story'leri tespit et
- Birden fazla Codex instance'ı paralel çalıştır
- Conflict resolution stratejisi

**prd.json Güncellemesi:**
```json
{
  "id": "US-003",
  "title": "Add UI component",
  "dependsOn": ["US-001", "US-002"],
  "passes": false
}
```

---

### 5. **Web UI ile PRD Yönetimi**
PRD oluşturma ve düzenleme için web arayüzü.

**Özellikler:**
- Drag-and-drop story sıralama
- Story ekleme/düzenleme/silme
- Acceptance criteria editörü
- Önizleme modu
- JSON export/import

**Teknik Uygulama:**
- Flowchart uygulamasına ekleme
- React Hook Form ile form yönetimi
- LocalStorage veya dosya sistemi kaydetme

---

## 🔧 Orta Öncelikli Özellikler

### 6. **GitHub Actions Entegrasyonu**
Ralph'ı CI/CD pipeline'ında çalıştırma.

**Özellikler:**
- `.github/workflows/ralph.yml` template'i
- PR oluşturma ve güncelleme
- Status check olarak çalışma
- Artifact olarak progress kaydetme

**Örnek Workflow:**
```yaml
name: Ralph Agent
on:
  workflow_dispatch:
    inputs:
      max_iterations:
        default: '10'

jobs:
  ralph:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Codex
        run: npm install -g codex-cli
      - name: Run Ralph
        env:
          CODEX_API_KEY: ${{ secrets.CODEX_API_KEY }}
          RALPH_CODEX_FULL_AUTO: 1
        run: ./ralph.sh ${{ inputs.max_iterations }}
```

---

### 7. **Çoklu PRD Desteği**
Birden fazla PRD dosyasını yönetme.

**Özellikler:**
- `prd/` klasöründe birden fazla JSON dosyası
- CLI ile PRD seçimi: `./ralph.sh --prd feature-x.json`
- PRD durumu raporlama
- Öncelik sıralaması

---

### 8. **Test Coverage Tracking**
Her story için test coverage takibi.

**Özellikler:**
- Coverage threshold'ları tanımlama
- Story tamamlandıktan sonra coverage raporu
- Coverage düşerse uyarı
- `progress.txt`'e coverage metrikleri ekleme

---

### 9. **Rollback Mekanizması**
Başarısız değişiklikleri geri alma.

**Özellikler:**
- Her story öncesi checkpoint
- Başarısız story'de otomatik rollback
- Manuel rollback komutu
- Git tag'leri ile versiyon takibi

---

### 10. **Context Window Optimizasyonu**
Daha verimli context kullanımı.

**Özellikler:**
- Story başına relevant dosyaları tespit et
- `AGENTS.md` dosyalarından bağlam çıkar
- Gereksiz dosyaları filtrele
- Context kullanımını logla

---

## 📊 Düşük Öncelikli / Gelecek Özellikler

### 11. **VS Code Extension**
Ralph'ı VS Code içinden kullanma.

**Özellikler:**
- PRD dosyası görselleştirme
- Inline story durumu
- Ralph başlatma/durdurma
- Progress log görüntüleme

---

### 12. **LLM Provider Agnostik Yapı**
Farklı LLM provider'ları destekleme.

**Özellikler:**
- OpenAI, Anthropic, Ollama desteği
- Environment variable ile seçim
- Model karşılaştırma istatistikleri

---

### 13. **Cost Tracking**
API kullanım maliyeti takibi.

**Özellikler:**
- Token kullanımı loglama
- Story başına maliyet hesaplama
- Budget limiti belirleme
- Maliyet raporu

---

### 14. **Template Library**
Yaygın PRD şablonları.

**Özellikler:**
- CRUD operasyonları template'i
- Authentication template'i
- API endpoint template'i
- UI component template'i

---

### 15. **Multi-Language Support**
Farklı dillerde prompt desteği.

**Özellikler:**
- Türkçe, İngilizce, Almanca prompt'lar
- Otomatik dil algılama
- Lokalize hata mesajları

---

## 🚀 Uygulama Öncelikleri

| Özellik | Etki | Zorluk | Öncelik |
|---------|------|--------|---------|
| Dashboard | Yüksek | Orta | 1 |
| Slack/Discord Bildirimleri | Yüksek | Düşük | 2 |
| Retry Mekanizması | Yüksek | Düşük | 3 |
| GitHub Actions | Yüksek | Düşük | 4 |
| Web UI PRD Yönetimi | Orta | Yüksek | 5 |
| Paralel Execution | Yüksek | Yüksek | 6 |
| Test Coverage | Orta | Orta | 7 |
| Rollback | Orta | Orta | 8 |

---

## 💡 Hızlı Kazanımlar (Quick Wins)

Hemen uygulanabilecek küçük iyileştirmeler:

1. **Renkli CLI Output**: Ralph iterasyonlarını renkli göster
2. **Progress Bar**: Story tamamlama durumunu progress bar ile göster
3. **Ses Bildirimi**: Tamamlandığında veya hata oluştuğunda sistem sesi
4. **Duration Logging**: Her story için geçen süreyi logla
5. **Summary Report**: Çalışma sonunda özet rapor

---

## 📝 Önerilen İlk Adımlar

1. **Slack Bildirimleri** - En düşük efor, yüksek değer
2. **Retry Mekanizması** - Güvenilirliği artırır
3. **Dashboard** - Görsellik ve kullanıcı deneyimi
4. **GitHub Actions Template** - CI/CD entegrasyonu

Bu özelliklerden hangilerini uygulamak isterseniz, detaylı implementasyon planı hazırlayabilirim.
