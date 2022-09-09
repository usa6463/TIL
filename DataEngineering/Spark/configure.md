## 설정 우선순위

In general, configuration values explicitly set on a SparkConf take the highest precedence, then flags passed to spark-submit, then values in the defaults file.

코드에 명시한 설정 >> spark-submit에 명시한 설정 >> default config 파일에 명시된 설정