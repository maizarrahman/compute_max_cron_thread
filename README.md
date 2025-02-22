# compute_max_cron_thread
Estimates Odoo max_cron_thread setting

Menghitung max_cron_thread

Baris ini akan di-conflict-kan

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

https://www.statology.org/pandas-group-by-5-minute-intervals/
```
import pandas as pd

#create DataFrame
df = pd.DataFrame({'date': pd.date_range(start='1/1/2020', freq='min', periods=12),
                   'sales': [6, 8, 9, 11, 13, 8, 8, 15, 22, 9, 8, 4],
                   'returns': [0, 3, 2, 2, 1, 3, 2, 4, 1, 5, 3, 2]})

#set 'date' column as index
df = df.set_index('date')

#view DataFrame
print(df)

                     sales  returns
date                               
2020-01-01 00:00:00      6        0
2020-01-01 00:01:00      8        3
2020-01-01 00:02:00      9        2
2020-01-01 00:03:00     11        2
2020-01-01 00:04:00     13        1
2020-01-01 00:05:00      8        3
2020-01-01 00:06:00      8        2
2020-01-01 00:07:00     15        4
2020-01-01 00:08:00     22        1
2020-01-01 00:09:00      9        5
2020-01-01 00:10:00      8        3
2020-01-01 00:11:00      4        2

#calculate sum of sales and returns grouped by 5-minute intervals
df.resample('5min').sum()

                     sales returns
date		
2020-01-01 00:00:00	47	 8
2020-01-01 00:05:00	62	15
2020-01-01 00:10:00	12 	 5
```
