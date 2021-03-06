```C++
#include <iostream>
#include "freeling.h"
#include "freeling/morfo/analyzer.h"
#include "freeling/output/output_conll.h"
using namespace std;


///////////////////////////////////////////////////
// Load an ad-hoc set of configuration options

freeling::analyzer::config_options fill_config(const wstring &lang, const wstring &ipath) {

  freeling::analyzer::config_options cfg;

  // Language of text to process
  cfg.Lang = lang;
 
  // path to language specific data
  wstring lpath = ipath + L"/share/freeling/" + cfg.Lang + L"/";

  // Tokenizer configuration file
  cfg.TOK_TokenizerFile = lpath + L"tokenizer.dat";
  // Splitter configuration file
  cfg.SPLIT_SplitterFile = lpath + L"splitter.dat";
  // Morphological analyzer options
  cfg.MACO_Decimal = L".";
  cfg.MACO_Thousand = L",";
  cfg.MACO_LocutionsFile = lpath + L"locucions.dat";
  cfg.MACO_QuantitiesFile = lpath + L"quantities.dat";
  cfg.MACO_AffixFile = lpath + L"afixos.dat";
  cfg.MACO_ProbabilityFile = lpath + L"probabilitats.dat";
  cfg.MACO_DictionaryFile = lpath + L"dicc.src";
  cfg.MACO_NPDataFile = lpath + L"np.dat";
  cfg.MACO_PunctuationFile = lpath + L"../common/punct.dat";
  cfg.MACO_ProbabilityThreshold = 0.001;

  // Sense annotator and WSD config files
  cfg.SENSE_ConfigFile = lpath + L"senses.dat";
  cfg.UKB_ConfigFile = lpath + L"ukb.dat";
  // Tagger options
  cfg.TAGGER_HMMFile = lpath + L"tagger.dat";
  cfg.TAGGER_ForceSelect = freeling::RETOK;
  // Statistical dependency parser & SRL config file
  cfg.DEP_TreelerFile = lpath + L"dep_treeler/dependences.dat";   

  // NEC config file. This module will not be loaded
  cfg.NEC_NECFile = L"";
  // Chart parser config file. This module will not be loaded
  cfg.PARSER_GrammarFile = L"";
  // Rule based dependency parser config files. This module will not be loaded
  cfg.DEP_TxalaFile = L"";
  // Coreference resolution config file. This module will not be loaded
  cfg.COREF_CorefFile = L"";

  return cfg;
}


///////////////////////////////////////////////////
// Load an ad-hoc set of invoke options

freeling::analyzer::invoke_options fill_invoke() {

  freeling::analyzer::invoke_options ivk;

  // Level of analysis in input and output
  ivk.InputLevel = freeling::TEXT;
  // We can not request higher analysis levels (e.g. coreference) because 
  // we didn't load the needed modules.
  // But we can use this option to lowe the analysis level at will during
  // our application execution.
  ivk.OutputLevel = freeling::DEP; 
  
  // activate/deactivate morphological analyzer modules
  ivk.MACO_UserMap = false;
  ivk.MACO_AffixAnalysis = true;
  ivk.MACO_MultiwordsDetection = true;
  ivk.MACO_NumbersDetection = true;
  ivk.MACO_PunctuationDetection = true;
  ivk.MACO_DatesDetection = true;
  ivk.MACO_QuantitiesDetection  = true;
  ivk.MACO_DictionarySearch = true;
  ivk.MACO_ProbabilityAssignment = true;
  ivk.MACO_CompoundAnalysis = false;
  ivk.MACO_NERecognition = true;
  ivk.MACO_RetokContractions = false;
    
  ivk.SENSE_WSD_which = freeling::UKB;
  ivk.TAGGER_which = freeling::HMM;

  // since we only created dep_treeler parser, we can not set the parser to use to another one.
  // If we had loaded both parsers, we could change the used parsed at will with this option
  ivk.DEP_which = freeling::TREELER;

  // since we did not load the module, setting this to true would trigger an error.
  // if the module was created, we could activate/deactivate it at will with this option.
  ivk.NEC_NEClassification = false; 

  return ivk;
}




/////////////   MAIN PROGRAM  /////////////////////

int main (int argc, char **argv) {

  // set locale to an UTF8 compatible locale 
  freeling::util::init_locale(L"default");

  // get requested language from arg1, or English if not provided
  wstring lang = L"en";
  if (argc > 1) lang = freeling::util::string2wstring(argv[1]);
  // get installation path to use from arg2, or use /usr/local if not provided
  wstring ipath = L"/usr/local";
  if (argc > 2) ipath = freeling::util::string2wstring(argv[2]);

  // set config options (which modules to create, with which configuration)
  freeling::analyzer::config_options cfg = fill_config(lang, ipath);
  // create analyzer
  freeling::analyzer anlz(cfg);

  // set invoke options (which modules to use. Can be changed in run time)
  freeling::analyzer::invoke_options ivk = fill_invoke();
  // load invoke options into analyzer
  anlz.set_current_invoke_options(ivk);

  // load document to analyze
  wstring text;  
  wstring line;
  while (getline(wcin,line)) 
    text = text + line + L"\n";

  // analyze text, leave result in ls
  list<freeling::sentence> ls;
  anlz.analyze(text,ls);

  // Create output handler and select desired output
  freeling::io::output_conll out(L"out1.cfg");
  // print analysis results in conll format
  out.PrintResults(wcout,ls);
}

```
