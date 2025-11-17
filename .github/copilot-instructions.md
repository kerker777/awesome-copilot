以下指引僅在進行程式碼審查時適用。

## README 更新

* [ ] 新文件應加入至 `README.md`。

## Prompt 文件指南

**僅適用於以 `.prompt.md` 結尾的文件**

* [ ] Prompt 包含 markdown front matter。
* [ ] Prompt 指定了 `mode` 欄位,值為 `agent` 或 `ask`。
* [ ] Prompt 包含 `description` 欄位。
* [ ] `description` 欄位不為空。
* [ ] `description` 欄位值以單引號包裹。
* [ ] 檔名為小寫,單字間以連字號分隔。
* [ ] 鼓勵使用 `tools`,但非必要。
* [ ] 強烈鼓勵使用 `model` 指定 prompt 最佳化的模型。

## Instruction 文件指南

**僅適用於以 `.instructions.md` 結尾的文件**

* [ ] Instruction 包含 markdown front matter。
* [ ] Instruction 包含 `description` 欄位。
* [ ] `description` 欄位不為空。
* [ ] `description` 欄位值以單引號包裹。
* [ ] 檔名為小寫,單字間以連字號分隔。
* [ ] Instruction 包含 `applyTo` 欄位,指定指引適用的檔案。如果需要指定多個檔案路徑,應格式化為 `'**.js, **.ts'`。

## Chat Mode 文件指南

**僅適用於以 `.chatmode.md` 結尾的文件**

* [ ] Chat mode 包含 markdown front matter。
* [ ] Chat mode 包含 `description` 欄位。
* [ ] `description` 欄位不為空。
* [ ] `description` 欄位值以單引號包裹。
* [ ] 檔名為小寫,單字間以連字號分隔。
* [ ] 鼓勵使用 `tools`,但非必要。
* [ ] 強烈鼓勵使用 `model` 指定 chat mode 最佳化的模型。
