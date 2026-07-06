# rust-book-okf

An **[OKF](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md) knowledge bundle** of
[The Rust Programming Language](https://doc.rust-lang.org/book/) book — ready
for agents to read and for you to chat with.

```bash
pip install okf-kit
okf get rust-book                     # via the awesome-okf-kit registry
okf chat rust-book --provider ollama  # or --provider openai
```

Or use it directly from this repo (the bundle is in `rust-book/`):

```bash
okf chat rust-book --provider ollama
okf visualize rust-book
```

Built and kept fresh with [okf-kit](https://github.com/vinodborole/okf-kit).
Content license: **MIT OR Apache-2.0** — see [NOTICE.md](NOTICE.md).
