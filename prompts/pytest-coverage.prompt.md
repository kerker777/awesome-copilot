---
agent: agent
description: '執行具有涵蓋率的 pytest 測試，探索遺失涵蓋率的行，並將涵蓋率提高到 100%。'
---

目標是讓測試涵蓋所有程式碼行。

使用以下指令產生涵蓋率報告：

pytest --cov --cov-report=annotate:cov_annotate

如果您要檢查特定模組的涵蓋率，可以這樣指定：

pytest --cov=your_module_name --cov-report=annotate:cov_annotate

您也可以指定要執行的特定測試，例如：

pytest tests/test_your_module.py --cov=your_module_name --cov-report=annotate:cov_annotate

開啟 cov_annotate 目錄以檢視已註解的原始碼。
每個原始碼檔案會有一個對應檔案。如果檔案具有 100% 的原始碼涵蓋率，表示所有行都由測試涵蓋，因此您不需要開啟該檔案。

對於測試涵蓋率低於 100% 的每個檔案，在 cov_annotate 中尋找相符的檔案並審查該檔案。

如果某行以 !（驚嘆號）開頭，表示該行未被測試涵蓋。
新增測試以涵蓋遺失的行。

持續執行測試並改善涵蓋率，直到所有行都被涵蓋。
