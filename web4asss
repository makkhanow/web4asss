using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Text.Json;

public static class AttendanceLog
{
    public static void Record(string path, string name)
    {
        string line = name + "," + DateTime.Now;
        File.WriteAllText(path, line);
    }

    public static string ReadAll(string path)
    {
        return File.ReadAllText(path);
    }
}

public record AttendanceEntry(string Name, DateTime CheckInTime)
{
    public static string ToLine(AttendanceEntry entry)
    {
        return $"{entry.Name},{entry.CheckInTime.ToString("o", CultureInfo.InvariantCulture)}";
    }

    public static AttendanceEntry FromLine(string line)
    {
        if (string.IsNullOrWhiteSpace(line))
            throw new ArgumentException("Line cannot be empty.", nameof(line));

        var parts = line.Split(',');
        if (parts.Length < 2)
            throw new FormatException($"Invalid line format: '{line}'");

        string name = parts[0];
        DateTime time = DateTime.Parse(parts[1], CultureInfo.InvariantCulture, DateTimeStyles.RoundtripKind);

        return new AttendanceEntry(name, time);
    }
}

public interface IAttendanceStore
{
    void Append(IEnumerable<AttendanceEntry> entries);
    IEnumerable<AttendanceEntry> Load();
}

public class TextLogStore : IAttendanceStore
{
    private readonly string _filePath;

    public TextLogStore(string filePath)
    {
        _filePath = filePath;
    }

    public void Append(IEnumerable<AttendanceEntry> entries)
    {
        if (entries == null) return;

        using (var writer = new StreamWriter(_filePath, append: true))
        {
            foreach (var entry in entries)
            {
                writer.WriteLine(AttendanceEntry.ToLine(entry));
            }
        }
    }

    public IEnumerable<AttendanceEntry> Load()
    {
        if (!File.Exists(_filePath))
        {
            return Enumerable.Empty<AttendanceEntry>();
        }

        var result = new List<AttendanceEntry>();
        using (var reader = new StreamReader(_filePath))
        {
            string? line;
            while ((line = reader.ReadLine()) != null)
            {
                if (!string.IsNullOrWhiteSpace(line))
                {
                    result.Add(AttendanceEntry.FromLine(line));
                }
            }
        }
        return result;
    }
}

public class JsonStore : IAttendanceStore
{
    private readonly string _filePath;
    private static readonly JsonSerializerOptions _options = new JsonSerializerOptions
    {
        WriteIndented = true
    };

    public JsonStore(string filePath)
    {
        _filePath = filePath;
    }

    public void Append(IEnumerable<AttendanceEntry> entries)
    {
        if (entries == null) return;

        var existingEntries = Load().ToList();
        existingEntries.AddRange(entries);

        string json = JsonSerializer.Serialize(existingEntries, _options);

        using (var writer = new StreamWriter(_filePath, append: false))
        {
            writer.Write(json);
        }
    }

    public IEnumerable<AttendanceEntry> Load()
    {
        if (!File.Exists(_filePath))
        {
            return Enumerable.Empty<AttendanceEntry>();
        }

        using (var reader = new StreamReader(_filePath))
        {
            string json = reader.ReadToEnd();
            if (string.IsNullOrWhiteSpace(json))
                return Enumerable.Empty<AttendanceEntry>();

            return JsonSerializer.Deserialize<List<AttendanceEntry>>(json, _options)
                   ?? new List<AttendanceEntry>();
        }
    }
}

class Program
{
    static void Main()
    {
        Console.WriteLine("=== Starter Code Diagnostics ===");
        string bugPath = "starter_bug.txt";
        if (File.Exists(bugPath)) File.Delete(bugPath);

        AttendanceLog.Record(bugPath, "Alice");
        AttendanceLog.Record(bugPath, "Bob");
        Console.WriteLine($"Starter Code Output: '{AttendanceLog.ReadAll(bugPath)}'");

        try
        {
            AttendanceLog.ReadAll("missing_file.txt");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Starter Code Exception: {ex.GetType().Name} - {ex.Message}");
        }

        Console.WriteLine("\n=== Single Store Demonstration ===");
        string demoTextPath = "demo_log.txt";
        string demoJsonPath = "demo_log.json";
        if (File.Exists(demoTextPath)) File.Delete(demoTextPath);
        if (File.Exists(demoJsonPath)) File.Delete(demoJsonPath);

        RunDemo(new TextLogStore(demoTextPath), "TextLogStore");
        RunDemo(new JsonStore(demoJsonPath), "JsonStore");

        Console.WriteLine("\n=== Multi-Run Simulation ===");
        string runTextPath = "sim_log.txt";
        string runJsonPath = "sim_log.json";
        if (File.Exists(runTextPath)) File.Delete(runTextPath);
        if (File.Exists(runJsonPath)) File.Delete(runJsonPath);

        IAttendanceStore textStore = new TextLogStore(runTextPath);
        IAttendanceStore jsonStore = new JsonStore(runJsonPath);

        Console.WriteLine("\n--- Run 1 ---");
        var run1Entries = new[]
        {
            new AttendanceEntry("John Doe", DateTime.Now.AddHours(-2)),
            new AttendanceEntry("Jane Miller", DateTime.Now.AddHours(-1))
        };
        SimulateRun(textStore, jsonStore, run1Entries);

        Console.WriteLine("\n--- Run 2 ---");
        var run2Entries = new[]
        {
            new AttendanceEntry("Alex Wong", DateTime.Now.AddMinutes(-30)),
            new AttendanceEntry("Emily Davis", DateTime.Now.AddMinutes(-10))
        };
        SimulateRun(textStore, jsonStore, run2Entries);
    }

    static void RunDemo(IAttendanceStore store, string storeName)
    {
        Console.WriteLine($"Running demo for: {storeName}");
        var entries = new[]
        {
            new AttendanceEntry("Demo User 1", DateTime.Now.AddMinutes(-10)),
            new AttendanceEntry("Demo User 2", DateTime.Now.AddMinutes(-5))
        };

        store.Append(entries);
        var loaded = store.Load().ToList();

        Console.WriteLine($"Items loaded: {loaded.Count}");
        foreach (var entry in loaded)
        {
            Console.WriteLine($" - {entry.Name} @ {entry.CheckInTime:HH:mm:ss}");
        }
    }

    static void SimulateRun(IAttendanceStore textStore, IAttendanceStore jsonStore, AttendanceEntry[] entries)
    {
        textStore.Append(entries);
        jsonStore.Append(entries);

        var textData = textStore.Load().ToList();
        var jsonData = jsonStore.Load().ToList();

        Console.WriteLine($"TextStore Count: {textData.Count} | JsonStore Count: {jsonData.Count}");
        Console.WriteLine($"Stores Match: {textData.SequenceEqual(jsonData)}");

        foreach (var entry in textData)
        {
            Console.WriteLine($"   {entry.Name} -> {entry.CheckInTime:HH:mm:ss}");
        }
    }
}