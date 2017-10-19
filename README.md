# for_isucon

## Makefile
便利コマンド群

```
make

```


## Systemdのサービス確認
一覧
```
systemctl list-units --type=service
```

設定ファイル場所
```
ls /etc/systemd/system/
```

ステータス確認
```
systemctl status isu-ruby
```

リスタート
```
systemctl restart isu-ruby
```

## alp
nginx のログ解析
ベンチ結果を解析してアプリケーションのどこにボトルネックが潜んでいるか見分ける

```
wget https://github.com/tkuchiki/alp/releases/download/v0.3.1/alp_linux_amd64.zip
```

```
sudo ./alp -r --sum -f /var/log/nginx/access.log
```

クエリストリングを考慮する場合
```
sudo ./alp -q -r --sum -f /var/log/nginx/access.log
```

## mysql接続
Sequel Pro で SSH 経由で接続
GUI なら index を試しに張ってみたりするの楽
コンソールで発行されたSQL見れて便利


## mysqldumpslow

```
general_log
general_log_file=/var/log/mysql/mysql.log
slow_query_log
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 0;
```

```
sudo mysqldumpslow /var/log/mysql/mysql-slow.log
```

## pt-query-digest

```
$ wget https://repo.percona.com/apt/percona-release_0.1-3.$(lsb_release -sc)_all.deb
$ sudo dpkg -i percona-release_0.1-3.$(lsb_release -sc)_all.deb
$ sudo apt-get install percona-toolkit
```

```
pt-query-digest /path/to/slow.log

やたら時間かかるからやめるかも

# サマリ

# 13.7s user time, 170ms system time, 45.35M rss, 125.62M vsz
# Current date: Tue Oct 17 22:07:25 2017
# Hostname: debian-8
# Files: /var/log/mysql/mysql-slow.log
# Overall: 99.69k total, 69 unique, 0.18 QPS, 0.00x concurrency __________
# Time range: 2017-10-11 14:09:50 to 2017-10-17 22:07:25
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time            36s     2us   151ms   363us   176us     2ms    11us
# Lock time          383ms       0    33ms     3us    10us   167us       0
# Rows sent        879.56k       0   9.77k    9.04    0.99  199.36       0
# Rows examine     156.57M       0  97.68k   1.61k    0.99  12.29k       0
# Query size         6.69M       6 953.59k   70.39   80.10   4.98k   31.70

# クエリをノーマライズして集計し降順に並べた表

# Profile
# Rank Query ID           Response time Calls R/Call V/M   Item
# ==== ================== ============= ===== ====== ===== ===============
#    1 0x16849282195BE09F 14.7475 40.7% 0.02 SELECT comments
#    2 0x7539A5F45EB76A80 11.7151 32.3%  5628 0.0021  0.01 SELECT comments
#    3 0x28FC5B5D583E2DA6  1.7237  4.8%  1521 0.0011  0.00 SHOW GLOBAL STATUS
#    4 0x3027353C36CAB572  1.6172  4.5%    41 0.0394  0.01 SELECT posts
#    5 0x99AA0165670CE848  1.5568  4.3% 31617 0.0000  0.00 ADMIN PREPARE
#    6 0xE070DA9421CA8C8D  1.0224  2.8% 19917 0.0001  0.00 SELECT users
#    7 0x44D0B06948A2E5CC  0.7879  2.2%   209 0.0038  0.02 SELECT posts
#    8 0x81F9D7F7FA80A53D  0.6532  1.8%    28 0.0233  0.00 SELECT comments
#    9 0xB718D440CBBB5C55  0.4962  1.4%   160 0.0031  0.00 SELECT posts
#   10 0x3A976F5EBE87A97C  0.4185  1.2%    22 0.0190  0.00 SELECT comments
# MISC 0xMISC              1.4838  4.1% 34943 0.0000   0.0 <59 ITEMS>

# これ以降は各クエリの詳細。Query ID で調べる

# Query 1: 87.50 QPS, 0.23x concurrency, ID 0x16849282195BE09F at byte 4879596
# Scores: V/M = 0.02
# Time range: 2017-10-11 14:10:55 to 14:11:59
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count          5    5600
# Exec time     40     15s     6us    95ms     3ms    18ms     7ms     8us
# Lock time      3    14ms       0   122us     2us    17us     7us       0
# Rows sent      0   2.06k       0       3    0.38    2.90    0.96       0
# Rows examine  46  73.06M       0  97.67k  13.36k  97.04k  33.34k       0
# Query size     6 449.55k      80      83   82.20   80.10    0.07   80.10
# String:
# Databases    isuconp
# Hosts        localhost
# Users        root
# Query_time distribution
#   1us  ################################################################
#  10us  ####################################
# 100us  #
#   1ms
#  10ms  ################
# 100ms
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'comments'\G
#    SHOW CREATE TABLE `isuconp`.`comments`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT * FROM `comments` WHERE `post_id` = 5845 ORDER BY `created_at` DESC LIMIT 3\G
```

## htop
ちょっとリッチなtop
CPU or IO、アプリケーション or DB などのあたりを付ける

```
sudo apt-get install htop
```

## rack-mini-profiler
https://github.com/MiniProfiler/rack-mini-profiler

ブラウザ左上にレスポンスと秒数が表示される。
ブラウザでポチポチしながら概要を把握するには便利。

## rack-lineprof
https://github.com/kainosnoema/rack-lineprof

ソース上に実際にかかった時間が表示される
これ自体が重いので検討をつけるときに使う


# せめかた
pt-query-digest で上位のクエリから sequel pro で explain まわしつつ、インデックス指定していく、めっちゃらく
