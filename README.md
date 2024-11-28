# compute_max_cron_thread
Estimates Odoo max_cron_thread setting

Menghitung max_cron_thread

1. Membuat dictionary jadwal dengan format: {"waktu": list_ids}
   Contoh data:
   {
       "2024-11-27 00:00:00": [1, 4, 7, 9, 10],
       "2024-11-27 00:00:01": [3, 6, 8],
       "2024-11-27 00:00:10": [2, 4],
       "2024-11-27 00:00:13": [5,],
   }

2. Cara mengisi dictionary tsb dan menentukan yg terbanyak. (Asumsi: prioritas dianggap sama semua)
   ```
   jadwal = {}
   batas_akhir = datetime.now().replace(microsecond=0) + timedelta(days=365)
   interval_detik = {
       'minutes': 60,
       'hours': 3600,
       'days': 3600*24,
       'weeks': 3600*24*7,
   }
   jadwal_terbanyak = None
   max_cron_thread = 2
   for cron_id in self.env['ir.cron'].search([('active', '=', True)]):
       waktu = cron_id.nextcall
       while waktu < batas_akhir:
          if jadwal.get(waktu):
              jadwal[waktu].append(cron_id.id)
              # Asumsi tiap proses selesai 1 detik
              if len(jadwal[waktu]) > max_cron_thread:
                  max_cron_thread = len(jadwal[waktu])
                  jadwal_terbanyak = waktu 
          else:
              jadwal[waktu] = [cron_id.id]
          if cron_id.interval_type != 'months':
              waktu += timedelta(seconds = cron_id.interval_number * interval_detik[cron_id.interval_type])
          else:
              waktu += cron_id.interval_number months
   ```
4. Menentukan max_cron_thread dalam interval sekian menit dan sekian jam

