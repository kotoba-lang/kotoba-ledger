(ns kotoba.ledger.file-git-test
  (:require [clojure.test :refer [deftest is testing]]
            [clojure.java.io :as io]
            [kotoba.ledger.core :as ledger]
            [kotoba.ledger.file-git :as fg])
  (:import [java.nio.file Files]
           [java.nio.file.attribute FileAttribute]))

(defn- temp-git-repo! []
  (let [dir (.toFile (Files/createTempDirectory "kotoba-ledger-test" (make-array FileAttribute 0)))]
    (let [pb (ProcessBuilder. ["git" "init" "-q"])]
      (.directory pb dir)
      (.waitFor (.start pb)))
    (let [pb (ProcessBuilder. ["git" "config" "user.email" "test@example.com"])]
      (.directory pb dir)
      (.waitFor (.start pb)))
    (let [pb (ProcessBuilder. ["git" "config" "user.name" "test"])]
      (.directory pb dir)
      (.waitFor (.start pb)))
    dir))

(deftest append-line-then-read-back
  (let [dir (temp-git-repo!)
        path (io/file dir "decisions.jsonl")
        event (ledger/proposal-event {:ts 1 :issue-id "i1" :proposal-id "p1"
                                       :kind :gmail/archive :risk :read-only :status :proposed})]
    (fg/append-line! path event)
    (is (= [event] (fg/read-events path)))))

(deftest append-and-commit-records-a-commit
  (let [dir (temp-git-repo!)
        path (io/file dir "decisions.jsonl")
        event (ledger/proposal-event {:ts 1 :issue-id "i1" :proposal-id "p1"
                                       :kind :gmail/archive :risk :read-only :status :proposed})]
    (fg/append-and-commit! dir "decisions.jsonl" event "record decision p1")
    (let [pb (ProcessBuilder. ["git" "log" "--oneline"])]
      (.directory pb dir)
      (.redirectErrorStream pb true)
      (let [proc (.start pb)
            out (slurp (.getInputStream proc))]
        (.waitFor proc)
        (is (re-find #"record decision p1" out))))))

(deftest append-line-creates-missing-parent-directories
  (testing "a path under a not-yet-existing subdirectory (e.g. data/proposals.jsonl
            in a fresh checkout) must not throw FileNotFoundException"
    (let [dir (temp-git-repo!)
          path (io/file dir "data/proposals.jsonl")
          event (ledger/proposal-event {:ts 1 :issue-id "i1" :proposal-id "p1"
                                         :kind :gmail/archive :risk :read-only :status :proposed})]
      (fg/append-line! path event)
      (is (= [event] (fg/read-events path))))))
