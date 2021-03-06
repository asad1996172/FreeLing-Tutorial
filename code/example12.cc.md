```C++
#include <iostream>
#include "freeling.h"
#include "freeling/morfo/analyzer.h"

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

  // other modules are not needed in this example
  // Sense annotator and WSD config files
  cfg.SENSE_ConfigFile = L"";
  cfg.UKB_ConfigFile = L"";
  // Tagger options
  cfg.TAGGER_HMMFile = L"";
  // Statistical dependency parser & SRL config file
  cfg.DEP_TreelerFile = L"";
  // NEC config file. 
  cfg.NEC_NECFile = L"";
  // Chart parser config file. 
  cfg.PARSER_GrammarFile = L"";
  // Rule based dependency parser config files. 
  cfg.DEP_TxalaFile = L"";
  // Coreference resolution config file.
  cfg.COREF_CorefFile = L"";

  return cfg;
}


///////////////////////////////////////////////////
// Load an ad-hoc set of invoke options

freeling::analyzer::invoke_options fill_invoke() {

  freeling::analyzer::invoke_options ivk;

  // Level of analysis in input and output
  ivk.InputLevel = freeling::TEXT;
  ivk.OutputLevel = freeling::MORFO; 
  
  // activate/deactivate morphological analyzer modules
  ivk.MACO_UserMap = false;
  ivk.MACO_AffixAnalysis = true;
  ivk.MACO_MultiwordsDetection = true;
  ivk.MACO_NumbersDetection = true;
  ivk.MACO_PunctuationDetection = true;
  ivk.MACO_DatesDetection = true;
  ivk.MACO_QuantitiesDetection  = true;
  ivk.MACO_DictionarySearch = true;
  ivk.MACO_CompoundAnalysis = false;
  ivk.MACO_NERecognition = true;
  ivk.MACO_RetokContractions = false;

  // deactivate guesser so words with no analysis do not get one
  // and can be processed by the alternative proposers
  ivk.MACO_ProbabilityAssignment = false;
    
  // other modules are not used in this examples
  ivk.NEC_NEClassification = false; 
  ivk.SENSE_WSD_which = freeling::NO_WSD;
  ivk.TAGGER_which = freeling::NO_TAGGER;
  ivk.DEP_which = freeling::NO_DEP;

  return ivk;
}


/////////////   MAIN PROGRAM  /////////////////////

int main (int argc, char **argv) {

  /// set locale to an UTF8 compatible locale
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

  // create alternative porposers
  freeling::alternatives alts_ort(ipath+L"/share/freeling/"+lang+L"/alternatives-ort.dat");
  // comment this out if there is no phonetic encoder for target language
  freeling::alternatives alts_phon(ipath+L"/share/freeling/"+lang+L"/alternatives-phon.dat");

  // load document to analyze
  wstring text;  
  wstring line;
  while (getline(wcin,line)) 
    text = text + line + L"\n";

  // analyze text, leave result in ls
  list<freeling::sentence> ls;
  anlz.analyze(text,ls);

  // propose alternative forms based on ortographical distance
  alts_ort.analyze(ls);
 
  // propose alternative forms based on phonetic distance
  // IMPORTANT: comment this out if there is no phonetic encoder for your language    
  alts_phon.analyze(ls);

  // print results, for each sentence
  for (list<freeling::sentence>::iterator s=ls.begin(); s!=ls.end(); s++) {
    // for each word
    for (freeling::sentence::iterator w=s->begin(); w!=s->end(); w++) {
      // print form, and analysis proposed by the morphological analyzer (if any)
      wcout<<L"FORM: "<<w->get_form()<<endl; 
      wcout<<L"   ANALYSIS:";
      for (freeling::word::iterator a=w->begin(); a!=w->end(); a++) 
        wcout<<L" ["<<a->get_lemma()<<L","<<a->get_tag()<<L"]";
      wcout<<endl;
      
      // print alternative forms proposed by the alternative suggestors
      wcout<<L"   ALTERNATIVE FORMS:";
      for (list<freeling::alternative>::iterator a=w->alternatives_begin(); a!=w->alternatives_end(); a++) 
        wcout<<L" ["<<a->get_form()<<L","<<a->get_distance()<<L"]";
      wcout<<endl; 
    }
    wcout<<endl;
  }
  
}

```
