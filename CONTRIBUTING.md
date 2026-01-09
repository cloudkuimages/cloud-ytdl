# Contributing to cloud-ytdl

Thank you for your interest in contributing to **cloud-ytdl**! We welcome bug reports, feature requests, and code contributions.

## 📋 Reporting Issues

We've set up issue templates to help you report bugs and suggest features effectively. When creating a new issue:

### Bug Reports
When reporting a bug, please include:
- **Clear description** of what went wrong
- **Steps to reproduce** the issue
- **Expected vs actual behavior**
- **Code example** demonstrating the problem
- **Environment details** (cloud-ytdl version, Node.js version, OS)
- **Error messages** or stack traces
- **Video URL** if the issue is specific to certain videos

### Feature Requests
When suggesting a feature:
- **Describe the problem** you're trying to solve
- **Explain your proposed solution**
- **Provide use cases** showing how this would be helpful
- **Include example code** if you have ideas on how the API should look

## 🔍 Before Reporting

Before creating a new issue, please:
1. **Search existing issues** to avoid duplicates
2. **Check the documentation** in the [README](README.md)
3. **Verify your Node.js version** (requires Node 18+)
4. **Try with the latest version** of cloud-ytdl

## 🐛 Common Issues and Solutions

### 403 Forbidden Errors
- This usually means the video requires authentication or is restricted
- **Solution:** Use cookies from your browser with `ytdl.createAgent(cookies)`
- See the [Downloading Restricted Videos](README.md#-downloading-restricted-videos) section

### Format Not Found
- The video may not have the specific format/quality you requested
- **Solution:** Use `ytdl.getInfo()` to check available formats

### Signature Decoding Errors
- YouTube occasionally changes their signature algorithm
- **Solution:** Update to the latest version of cloud-ytdl or provide valid cookies

## 💻 Code Contributions

We welcome pull requests! If you'd like to contribute code:

1. **Fork the repository** and create a new branch
2. **Make your changes** with clear, focused commits
3. **Test your changes** thoroughly
4. **Submit a pull request** with a clear description

## 📝 Code Style

- Follow existing code style and conventions
- Use modern JavaScript (ES6+) features
- Add comments for complex logic
- Keep functions focused and modular

## ⚠️ Responsible Use

Please use this library in accordance with YouTube's Terms of Service. The library is intended for:
- Personal video archival
- Accessibility (downloading videos for offline viewing)
- Educational purposes
- Content creation and analysis

**Not recommended for:**
- Automated large-scale downloads
- Redistributing copyrighted content
- Violating YouTube's Terms of Service

## 📞 Getting Help

- **Documentation:** Start with the [README](README.md) for comprehensive API docs
- **Issues:** Report bugs or request features via GitHub Issues
- **Discussions:** Ask questions in GitHub Discussions
- **Examples:** Check the README for usage examples

## 🙏 Thank You!

Your contributions help make cloud-ytdl better for everyone. Whether you're reporting a bug, suggesting a feature, or contributing code, we appreciate your support!

---

**Author:** [AlfiDev](https://github.com/cloudkuimages)  
**Repository:** [github.com/cloudkuimages/cloud-ytdl](https://github.com/cloudkuimages/cloud-ytdl)
