#include <iostream>
#include <string>
#include <regex>
#include <unordered_map>
#include <vector>
#include <algorithm>

using namespace std;

// Перелічення типів токенів
enum class TokenType {
    NUMBER,
    STRING_LITERAL,
    CHAR_LITERAL,
    PREPROCESSOR,
    COMMENT,
    KEYWORD,
    OPERATOR,
    DELIMITER,
    IDENTIFIER,
    UNKNOWN
};

// Структура токена
struct Token {
    string value;
    TokenType type;
};

// Ключові слова C++
const vector<string> keywords = {
    "int", "float", "double", "return", "if", "else",
    "for", "while", "do", "void", "cout", "cin"
};

// Перевірка, чи слово є ключовим
bool isCppKeyword(const string& word) {
    return find(keywords.begin(), keywords.end(), word) != keywords.end();
}

// Визначення типу токена
TokenType detectTokenType(const string& token) {
    if (regex_match(token, regex(R"(\d+(\.\d+)?([eE][+\-]?\d+)?)"))) return TokenType::NUMBER;
    if (regex_match(token, regex(R"("([^"\\]|\\.)*")"))) return TokenType::STRING_LITERAL;
    if (regex_match(token, regex(R"('([^'\\]|\\.)*')"))) return TokenType::CHAR_LITERAL;
    if (regex_match(token, regex(R"(#\w+)"))) return TokenType::PREPROCESSOR;
    if (regex_match(token, regex(R"(//.*)"))) return TokenType::COMMENT;
    if (regex_match(token, regex(R"(/\*[\s\S]*?\*/)", regex::icase))) return TokenType::COMMENT;
    if (isCppKeyword(token)) return TokenType::KEYWORD;
    if (regex_match(token, regex(R"([\+\-\*\/=<>!&|]+)"))) return TokenType::OPERATOR;
    if (regex_match(token, regex(R"([{}()\[\];,])"))) return TokenType::DELIMITER;
    if (regex_match(token, regex(R"([a-zA-Z_][a-zA-Z0-9_]*)"))) return TokenType::IDENTIFIER;
    return TokenType::UNKNOWN;
}

// Основна функція аналізу коду
void tokenizeSourceCode(const string& code) {
    unordered_map<TokenType, vector<Token>> tokenGroups;

    regex tokenRegex(
        R"(\s+|//.*|/\*[\s\S]*?\*/|".*?"|'.*?'|#\w+|[a-zA-Z_][a-zA-Z0-9_]*|\d+(\.\d+)?([eE][+\-]?\d+)?|[\+\-\*\/=<>!&|]+|[{}()\[\];,])"
    );

    auto begin = sregex_iterator(code.begin(), code.end(), tokenRegex);
    auto end = sregex_iterator();

    for (auto i = begin; i != end; ++i) {
        string tokenValue = i->str();
        if (regex_match(tokenValue, regex(R"(\s+)"))) continue; // пропустити пробіли
        TokenType type = detectTokenType(tokenValue);
        tokenGroups[type].push_back({ tokenValue, type });
    }

    // Вивід результатів
    for (const auto& pair : tokenGroups) {
        cout << "=== Token Type " << static_cast<int>(pair.first) << " ===\n";
        for (const auto& token : pair.second) {
            cout << "  " << token.value << "\n";
        }
        cout << endl;
    }
}

// Тестовий приклад
int main() {
    string codeExample = R"(
        #include <iostream>
        using namespace std;

        int main() {
            int a = 10;
            float b = 2.5;
            char c = 'Z';
            float result = a * b;
            if (result > 10) {
                cout << "Result: " << result << endl;
            } else {
                cout << "Too small" << endl;
            }
            return 0;
        }
    )";

    tokenizeSourceCode(codeExample);
    return 0;
}
