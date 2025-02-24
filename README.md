C# Example: ASP.NET Core File API
This sample controller demonstrates how to:

Read a file asynchronously: using ReadAllTextAsync to avoid blocking threads.
Handle errors gracefully: checking if the file exists and using try/catch with logging.
Use dependency injection: for configuration and logging.
csharp
Copy
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Logging;
using System;
using System.IO;
using System.Threading.Tasks;

namespace MyApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class FileController : ControllerBase
    {
        private readonly ILogger<FileController> _logger;
        private readonly string _filePath;

        public FileController(ILogger<FileController> logger, IConfiguration configuration)
        {
            _logger = logger;
            _filePath = configuration.GetValue<string>("FilePath") 
                        ?? throw new ArgumentNullException("FilePath not configured.");
        }

        [HttpGet]
        public async Task<IActionResult> GetFileContent()
        {
            if (!System.IO.File.Exists(_filePath))
            {
                _logger.LogError("File not found at path: {FilePath}", _filePath);
                return NotFound("File not found.");
            }

            try
            {
                var content = await System.IO.File.ReadAllTextAsync(_filePath);
                return Ok(content);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error reading file.");
                return StatusCode(500, "Internal server error.");
            }
        }
    }
}
Key points:

Configuration: The file path is injected via configuration so that it isn’t hardcoded.
Async Operations: The use of await ensures the API remains responsive.
Error Handling: Logs and returns appropriate HTTP responses based on success or failure.
React Example: Consuming the File API
This React component shows how to:

Fetch data asynchronously: using the fetch API within useEffect.
Manage state with hooks: using useState for both the content and error state.
Display content cleanly: rendering a <pre> tag to preserve formatting for text files.
jsx
Copy
import React, { useEffect, useState } from 'react';

const FileContent = () => {
  const [content, setContent] = useState('');
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchFileContent = async () => {
      try {
        const response = await fetch('/api/file');
        if (!response.ok) {
          throw new Error(`Error: ${response.status} ${response.statusText}`);
        }
        const text = await response.text();
        setContent(text);
      } catch (err) {
        setError(err.message);
      }
    };

    fetchFileContent();
  }, []);

  return (
    <div>
      <h1>File Content</h1>
      {error ? (
        <div style={{ color: 'red' }}>Error: {error}</div>
      ) : (
        <pre>{content}</pre>
      )}
    </div>
  );
};

export default FileContent;
Key points:

Separation of Concerns: The component handles fetching and state, while the server deals with file I/O.
Error Handling: Errors during the fetch are caught and rendered for user feedback.
Hooks: useEffect ensures the data is fetched once when the component mounts.
