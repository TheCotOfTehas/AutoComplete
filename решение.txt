using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using System.Text.RegularExpressions;

namespace AutoComplete
{

    public struct FullName
    {
        public string Name;
        public string Surname;
        public string Patronymic;
    }

    public class AutoCompleter
    {
        public static List<string> completions = new List<string>();
        private const char Separator = ' ';
        public void AddToSearch(List<FullName> fullNames)
        {
            completions = fullNames
                .Select(full => Concatenate(full))
                .Distinct()
                .Where(full => !string.IsNullOrWhiteSpace(full))
                .OrderBy(x => x)
                .ToList();
        }

        public static string Concatenate(FullName fullName)
        {
            var result = string.Empty;

            result += fullName.Surname?.Trim();
            result += Separator + fullName.Name?.Trim();
            result += Separator + fullName.Patronymic?.Trim();

            return result.Trim();
        }

        public int GetLeftBorderIndex(List<string> phrases, string prefix, int left, int right)
        {
            if ((right == left || right == 0) && left - 1 < -1) return left;
            if (right == left || right == 0) return left - 1;
            int mid = left / 2 + right / 2;
            var d = phrases[mid];
            if (String.Compare(d, prefix) >= 0)
            {
                return GetLeftBorderIndex(phrases, prefix, left, mid);
            }
            else
            {
                return GetLeftBorderIndex(phrases, prefix, mid + 1, right);
            }
        }

        //Метод принимает на вход строку по которой необходимо найти ФИО.
        public List<string> Search(string prefix)
        {
            if (string.IsNullOrWhiteSpace(prefix))
            {
                throw new ArgumentException("Значение prefix не имеет занчения либо состоит из одних пробелов", nameof(prefix));
            }

            if (prefix.Length > 100)
            {
                throw new ArgumentException("Значение prefix не может быть более 100 символов", nameof(prefix));
            }

            if (completions == null)
            {
                return new List<string>();
            }

            var LeftBorderIndex = GetLeftBorderIndex(completions, prefix, -1, completions.Count);
            var result = new List<string>();

            for (int i = LeftBorderIndex + 1; i < completions.Count; i++)
            {
                if (completions[i].StartsWith(prefix))
                {
                    result.Add(completions[i]);
                }
                else
                {
                    break;
                }
            }

            return result;
        }
    }
}